# 添加Filter

> **重要程度：高**｜**手练：需要**
> **重点**：Servlet 过滤链、放行、响应头与异常路径。

## Filter 解决什么问题

Servlet Filter 在请求到达目标处理器前后参与处理，适合统一请求日志、编码、必要的安全检查等。过滤器通过 `chain.doFilter` 调用后续链；不调用它就会短路，需要自己给出适当响应。

本例把 `Filter` 注册为 Spring Bean，为响应添加固定演示头，再放行请求。业务接口返回 JSON，过滤器不读取或修改请求体。Boot 4 使用 Jakarta Servlet 的 `jakarta.servlet` 包，旧教程中的 `javax.servlet` 不能直接按新版本照搬。

## 完整工程

使用 JDK 17、Maven 与 Spring Boot 4.1.2。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>filter-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

将代码保存为 `src/main/java/learn/filter/FilterDemo.java`：

```java
package learn.filter;
import java.util.Map;
import jakarta.servlet.Filter;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
@SpringBootApplication
@RestController
public class FilterDemo {
    private static final Logger logger = LoggerFactory.getLogger(FilterDemo.class);
    @Bean Filter demoFilter() {
        return (request, response, chain) -> {
            /* ========================================================================
             * 步骤1：过滤 HTTP 请求
             * 数据源：进入 Servlet 过滤链的请求和响应
             * 操作：1) 添加演示响应头 2) 继续调用并记录结束
             * ======================================================================== */
            logger.info("开始过滤请求");
            try {
                // 1.1 响应尚未提交时添加固定演示头
                ((HttpServletResponse) response).setHeader("X-Demo-Filter", "applied");
                // 1.2 放行到后续过滤器与接口
                chain.doFilter(request, response);
            } finally {
                logger.info("过滤请求结束");
            }
        };
    }
    @GetMapping("/hello") public Map<String, String> hello() {
        return Map.of("message", "Hello");
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动过滤器示例
         * 数据源：启动参数与 Boot 自动配置
         * 操作：1) 启动容器 2) 注册过滤器并接收请求
         * ======================================================================== */
        logger.info("开始启动过滤器示例");
        // 1.1 启动嵌入式服务器
        SpringApplication.run(FilterDemo.class, args);
        logger.info("过滤器示例启动完成");
    }
}
```

## 启动并请求

确认 `java -version` 与 `mvn -v` 使用 JDK 17 或以上。在工程目录运行：

```powershell
mvn package
java -jar target/filter-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell 请求：

```powershell
curl.exe -i http://127.0.0.1:8080/hello
```

应收到 HTTP 200，响应头包含 `X-Demo-Filter: applied`，JSON 中 `message` 为 `Hello`。结束后回到服务窗口按 `Ctrl+C`；若端口被占用，修改启动参数和请求地址。

## 调用顺序与边界

请求进入过滤器，先设置响应头，再调用 `chain.doFilter` 进入后续链与控制器。控制器返回后，执行 `finally` 中的结束日志；下游抛异常时 `finally` 也执行，异常继续向外传播。

过滤器顺序和适用路径需明确配置；多个过滤器的前后顺序会影响认证、日志和响应。异步和错误分发可能再次经过过滤器，是否需要每次执行一次应按具体注册和框架契约决定。不要无条件读取请求体，否则后续处理可能无法再次读取；不要记录密码、令牌等敏感内容。

Filter 位于 Servlet 链；Spring MVC Interceptor 位于 MVC 请求处理流程，两者覆盖范围和扩展点不同。安全敏感逻辑应优先使用项目的成熟安全机制，而不是只依赖这个演示过滤器。

## 动手与自查

1. 请求接口并检查状态码、JSON 与响应头。
2. 在隔离示例中暂时不调用 `chain.doFilter`，观察控制器是否执行，并自己设置明确的响应状态。
3. 让控制器抛异常，观察 `finally` 日志；解释为什么不能直接吞掉异常。

来源：[教程：添加Filter](https://liaoxuefeng.com/books/java/springboot/filter/index.html)、[Spring Boot Servlet 过滤器注册](https://docs.spring.io/spring-boot/reference/web/servlet.html)。
