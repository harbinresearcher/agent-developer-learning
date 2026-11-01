# 集成Filter

> **重要程度：高**｜**手练：需要**
> **重点**：容器过滤链、Spring Bean 注册、顺序与响应边界。

## Filter 如何进入 Spring 应用

Filter 属于 Servlet 容器的请求处理链；在 Boot 的 Servlet Web 应用中，注册为 Bean 的 Filter 可以参与容器过滤。需要指定 URL 模式、顺序或分发类型时，可按项目版本使用 `FilterRegistrationBean` 明确注册。

本例注册一个 Filter Bean，在下游处理前添加响应头，并调用 `chain.doFilter` 放行。接口只是固定 JSON；示例没有实现身份认证。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。先确认 `java -version`、`mvn -v` 均使用 JDK 17 或以上。新建目录并保存 `pom.xml`：

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

保存 `src/main/java/learn/filter/FilterDemo.java`：

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

工程目录运行：

```powershell
mvn package
java -jar target/filter-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/hello
```

应看到 HTTP 200、`X-Demo-Filter: applied` 响应头，以及包含 `message: Hello` 的 JSON。结束后在服务窗口按 `Ctrl+C`；端口被占用时同步修改启动参数与请求地址。

## 与 MVC 和安全链的关系

请求经过 Servlet 过滤链后才可能进入 Spring MVC 的处理器链；Filter 与 `HandlerInterceptor` 位于不同层次。实际顺序受注册、URL 模式、分发类型和安全框架配置影响，需要通过集成测试确认，不应只按注解出现的顺序判断。

示例在下游执行前设置头部，并在 `finally` 中记录结束；下游异常不会被示例吞掉。不要记录密码、令牌或完整敏感正文，也不要在过滤器中无界缓存上传数据。Spring Security 项目应遵循其过滤链配置，避免把独立过滤器当成完整的认证与授权。

## 动手与自查

1. 检查接口响应头，再让接口抛异常，观察过滤器结束日志。
2. 使用 `FilterRegistrationBean` 将路径限制为 `/hello`，并验证另一路径是否被过滤。
3. 为什么 Filter 在 Servlet 层，而 MVC Interceptor 不应被当成完全等价的替代？

来源：[教程：集成Filter](https://liaoxuefeng.com/books/java/spring/web/filter/index.html)、[Spring Boot Servlet Web 应用](https://docs.spring.io/spring-boot/reference/web/servlet.html)。
