# Web基础

> **重要程度：高**｜**手练：需要**
> **重点**：URL、HTTP 请求与响应、方法、状态码、无状态和接口验证。

## 一个请求发生了什么

客户端访问 `http://127.0.0.1:8080/hello`：`http` 是协议，`127.0.0.1` 是本机回环地址，`8080` 是端口，`/hello` 是路径。客户端向服务器发出 HTTP 请求，服务器根据方法和路径处理，再返回状态码、响应头和可选正文。

以 HTTP/1.1 文本形式理解，下列示意省略部分头部，实际 HTTP/2、HTTP/3 的传输格式不同：

```text
GET /hello HTTP/1.1
Host: 127.0.0.1:8080

HTTP/1.1 200 OK
Content-Type: application/json

{"message":"Hello, Spring Boot!"}
```

请求行包含方法、目标和协议版本；`Host` 指定目标主机；空行分隔头部与正文。响应中的 `200` 表示成功，`Content-Type` 告诉客户端正文类型。不要把 URL 参数、HTTP 头和请求正文当成同一位置。

## 完整练习：启动一个本地接口

本例使用 JDK 17、Maven 和 Spring Boot 4.1.2。先用 `java -version`、`mvn -v` 确认两者使用 JDK 17 或以上。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>hello-boot</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/boot/HelloApplication.java`：

```java
package learn.boot;

import java.util.Map;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class HelloApplication {
    private static final Logger logger = LoggerFactory.getLogger(HelloApplication.class);

    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动应用
         * 数据源：启动参数与 Spring Boot 自动配置
         * 操作：1) 创建应用容器 2) 启动内嵌 Web 服务
         * ======================================================================== */
        logger.info("开始启动应用");
        // 1.1 根据类路径和配置启动容器
        SpringApplication.run(HelloApplication.class, args);
        logger.info("应用启动完成");
    }

    @GetMapping("/hello")
    public Map<String, String> hello() {
        /* ========================================================================
         * 步骤1：处理问候请求
         * 数据源：GET /hello
         * 操作：1) 构造响应 2) 返回 JSON 数据
         * ======================================================================== */
        logger.info("开始处理问候请求");
        // 1.1 构造固定响应，不访问外部服务
        Map<String, String> result = Map.of("message", "Hello, Spring Boot!");
        logger.info("问候请求处理完成");
        return result;
    }
}
```

在工程目录运行：

```powershell
mvn package
java -jar target/hello-boot-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

保持服务运行，另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/hello
```

应收到 HTTP 200，JSON 的 `message` 为 `Hello, Spring Boot!`。`-i` 让 curl 显示响应头；服务端日志和 HTTP 响应是不同内容。结束后在服务窗口按 `Ctrl+C`。8080 被占用时，启动参数和请求地址使用同一个其他空闲端口。

## 常见方法和状态

- `GET` 通常读取资源；`POST` 通常提交数据或触发处理；`PUT` 通常替换目标资源的状态；`DELETE` 请求删除。方法表达协议语义，实际接口是否遵守要看服务实现。
- 幂等指重复同样请求的预期服务器效果与一次相同，不要求每次响应完全相同。`GET`、`PUT`、`DELETE` 的协议语义是幂等；`POST` 不保证幂等。重试付款等操作仍需服务端设计幂等机制。
- `2xx` 通常表示成功；`400` 常见于请求无效；`401` 表示缺少或未通过认证；`403` 表示服务器拒绝请求；`404` 表示目标未找到或不公开；`5xx` 表示服务器处理失败。具体错误格式由接口约定。

HTTP 是无状态协议：单个请求的语义不依赖服务器自动记住先前请求。应用可借助 Cookie、Session 或令牌关联身份和业务状态。HTTPS 在 HTTP 与传输层之间使用 TLS，提供传输保护，但不能代替身份授权、输入校验或安全的密钥管理。

## 工作中怎么用

接口设计需明确路径、方法、参数位置、状态码和错误格式。敏感信息不要放在 URL 或日志里；客户端应合理设置连接与请求超时。跨服务重试需判断操作是否幂等，不能认为网络超时就一定没有执行。

浏览器开发者工具的 Network 面板可查看请求方法、状态码、请求头与响应。浏览器显示的内容不一定等于服务端实际日志；排查时要核对双方以及中间代理的行为。

## 动手与自查

1. 用 `curl.exe -i` 查看响应头，找出状态码和内容类型。
2. 将路径改为不存在的 `/missing`，观察状态码；再对 `/hello` 使用 POST，观察方法不匹配的结果。
3. 为什么两次 `DELETE` 可能返回不同状态码，但仍可符合幂等语义？

来源：[教程：Web基础](https://liaoxuefeng.com/books/java/web/basic/index.html)、[HTTP 方法](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)、[幂等](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent)。
