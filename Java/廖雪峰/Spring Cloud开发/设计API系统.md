# 设计 API 系统

> **重要程度：高**｜**手练：需要**
> **重点**：认证与授权；请求校验和状态码；请求超时后的幂等重试；进程内演示与生产保障的区别。

## API 接收请求后做什么

交易 API 面向客户端接收下单、撤单和查询请求。典型处理链是：

```text
请求 → 认证身份 → 校验权限和参数 → 检查幂等键 → 提交业务命令 → 返回结果
```

认证回答“是谁”；授权回答“这个身份能否操作这笔订单或账户”。用户 ID 不应由请求体任意指定后直接信任，服务端要从经过验证的身份中获取，并校验资源归属。参数校验包括数量、价格、精度、交易对、请求大小和状态等。

API 收到请求不等于撮合已经完成。若命令只是被接受，需要给客户端查询入口和清晰的状态，不应把“已接受”展示成“已成交”。客户端超时也无法判断服务是否已接收；查询或重试时应使用原业务请求 ID。

## 手练：本机演示认证与请求去重

这是一个独立的 Spring Boot 示例，所有数据只在一个进程内保存。它用固定演示令牌模拟身份检查，**没有实现真正的用户认证、授权、交易、持久化或跨实例去重**；只绑定本机，不能用于生产。

目录为 `cloud-api-demo/pom.xml` 和 `cloud-api-demo/src/main/java/learn/cloudapi/ApiDemo.java`。

`pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>cloud-api-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

`src/main/java/learn/cloudapi/ApiDemo.java`：

```java
package learn.cloudapi;
import java.util.concurrent.ConcurrentHashMap;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class ApiDemo {
    private static final Logger logger = LoggerFactory.getLogger(ApiDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动本地 API 演示
         * 数据源：本机 HTTP 请求和演示配置
         * 操作：1) 加载配置 2) 启动应用
         * ======================================================================== */
        logger.info("开始启动 API 演示");
        // 1.1 启动 Spring Boot 应用
        SpringApplication.run(ApiDemo.class, args);
        logger.info("API 演示启动结束");
    }
}
record OrderRequest(String requestId, int quantity) {}
record OrderResult(String requestId, int quantity, String status) {}
@RestController
class OrderController {
    private static final Logger logger = LoggerFactory.getLogger(OrderController.class);
    private final ConcurrentHashMap<String, OrderResult> orders = new ConcurrentHashMap<>();
    private final String demoToken;
    OrderController(@Value("${demo.token}") String demoToken) { this.demoToken = demoToken; }
    @PostMapping("/orders")
    ResponseEntity<?> create(@RequestHeader(value = "Authorization", required = false) String authorization,
                             @RequestBody OrderRequest input) {
        /* ========================================================================
         * 步骤1：验证演示身份和请求
         * 数据源：授权头与下单请求体
         * 操作：1) 校验演示令牌 2) 校验请求编号和数量
         * ======================================================================== */
        logger.info("开始处理演示订单");
        // 1.1 未认证或参数无效时不创建订单
        if (!("Bearer " + demoToken).equals(authorization)) {
            logger.info("演示认证失败");
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).header("WWW-Authenticate", "Bearer").body("unauthorized");
        }
        if (input.requestId() == null || input.requestId().isBlank()
                || input.requestId().length() > 64 || input.quantity() <= 0) {
            logger.info("订单参数无效");
            return ResponseEntity.badRequest().body("invalid request");
        }
        logger.info("请求验证结束");
        /* ========================================================================
         * 步骤2：在演示进程内处理重复请求
         * 数据源：请求编号及内存订单表
         * 操作：1) 原子登记首次请求 2) 检查重复请求内容
         * ======================================================================== */
        logger.info("开始登记演示订单");
        // 2.1 相同请求编号原子地只创建一个内存对象
        OrderResult candidate = new OrderResult(input.requestId(), input.quantity(), "ACCEPTED");
        OrderResult prior = orders.putIfAbsent(input.requestId(), candidate);
        if (prior == null) {
            logger.info("演示订单创建结束");
            return ResponseEntity.status(HttpStatus.CREATED).body(candidate);
        }
        if (prior.quantity() != input.quantity()) {
            logger.info("请求编号冲突");
            return ResponseEntity.status(HttpStatus.CONFLICT).body("requestId conflict");
        }
        logger.info("返回已有演示订单");
        return ResponseEntity.ok(prior);
    }
}
```

项目根目录运行：

```powershell
mvn package
java -jar target/cloud-api-demo-1.0.jar --server.address=127.0.0.1 --server.port=18080 --demo.token=local-demo
```

另开 PowerShell：

```powershell
$headers = @{ Authorization = 'Bearer local-demo' }
$body = '{"requestId":"a1","quantity":2}'
Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:18080/orders' -Headers $headers -ContentType 'application/json' -Body $body
```

首次请求返回 HTTP 201，响应包含 `requestId=a1`、`quantity=2` 和 `status=ACCEPTED`；再发送完全相同的请求返回 200 与相同结果。同一请求 ID 改为数量 3 返回 409；错误令牌返回 401；空请求 ID 返回 400。

已用 JDK 17、Spring Boot 4.1.2 和隔离本机端口验证这五条 HTTP 路径。示例中的 `ACCEPTED` 仅表示对象写入演示进程内存，没有发送到定序器。Ctrl+C 停止服务后数据消失。

`putIfAbsent` 使**同一个进程内**同一编号只保存一个对象，但不能保证跨实例、跨重启或数据库与消息发布之间的幂等。生产实现应以经过认证的用户和业务操作限定幂等键范围，保存请求摘要、结果和状态，用持久化唯一约束处理并发，并设计处理中、成功、失败和过期规则。

## 认证与教程内容更新

- HTTP Basic 的凭证是 `用户名:密码` 的 Base64 编码，Base64 只是编码，不是加密；传输凭证时需要 HTTPS，且要控制凭证保管、轮换与日志泄露。
- Bearer 令牌被持有者即可使用，因此也需要安全传输、有效期、权限范围和撤销策略。上例固定令牌只用于本机理解请求头，不代表完整令牌认证。
- 教程说“API 不能使用 Session”过于绝对。API 可以使用 Session 与 Cookie，但要设计集群会话共享、Cookie 属性、跨域和 CSRF 防护；也可以采用无状态令牌。选择取决于客户端和安全架构。
- 如果把身份放在 `ThreadLocal`，线程复用时必须清理；异步处理、线程池和跨服务调用还需要显式处理上下文传播。生产系统应优先使用经过配置的安全框架，不要照搬本例手写认证。

## 工作中检查什么

1. **身份与权限**：登录、调用方身份、接口权限和资源归属都要核对；敏感操作需要审计与限流。
2. **重复请求**：用户网络超时后可能重试。对下单、转账等操作保留稳定请求 ID，返回已存在结果；同一 ID 内容不同应报冲突。
3. **可靠提交**：写数据库和向消息系统发事件不天然处于同一事务。根据业务选用 outbox、重试和对账，并确保消费端幂等。
4. **调用失败**：设置超时、有限重试和错误响应；不要把内部异常堆栈、密码或令牌返回给客户端或写入日志。
5. **观察与保护**：记录请求追踪 ID、业务 ID、状态码和耗时；控制请求体大小、限流和容量。追踪 ID 不是业务幂等键。

## 自测

1. 同一用户重试下单，为什么应复用原请求 ID？
2. 第一次请求已经成功但响应丢失时，重试如何区分重复与新订单？
3. `ConcurrentHashMap` 不能跨进程去重，服务重启后示例会怎样？
4. 登录成功后，为什么仍要检查当前用户是否有权查询指定订单？

来源：[原教程](https://liaoxuefeng.com/books/java/springcloud/api/index.html)；[HTTP 认证](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication)；[OWASP REST 安全](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)。
