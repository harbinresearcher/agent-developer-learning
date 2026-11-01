# 使用Filter

> **重要程度：高**｜**手练：需要**
> **重点**：过滤链、放行与短路、顺序、异常路径和响应提交。

## Filter 在哪里

Servlet Filter 位于容器请求处理链中。请求先进入一个或多个过滤器，再进入目标 Servlet；目标返回后，调用栈依次回到过滤器。它适合记录经过时间、补充响应头或交给成熟安全框架做统一认证等横切处理。

调用 `chain.doFilter(request, response)` 会继续后续链；不调用就会短路，过滤器需要自己产生合适的响应。过滤器不是控制器，不应把所有业务规则都堆进去。

## 完整例子

使用 JDK 17、Maven、Spring Boot 4.1.2 与 Jakarta Servlet。新建目录，保存 `pom.xml`：

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

先检查 `java -version` 和 `mvn -v` 均使用 JDK 17 或以上，再在工程目录运行：

```powershell
mvn package
java -jar target/filter-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/hello
```

应得到 HTTP 200、响应头 `X-Demo-Filter: applied`，JSON 中 `message` 为 `Hello`。服务日志显示过滤开始和结束；结束后在服务窗口按 `Ctrl+C`。端口被占用时应同步修改启动参数与请求地址。

## 跟踪调用顺序

请求进入过滤器，先设置响应头，再调用 `chain.doFilter`；下游控制器返回后才回到 `finally`。如果下游抛异常，`finally` 仍运行，异常继续传播。设置响应头要考虑响应是否已经提交；本例在调用下游前设置。

多个过滤器会按注册的顺序组成链，返回时按调用栈逆序。顺序会影响认证、跨域、日志和错误处理；应使用项目的明确配置并在真实链路测试。异步或错误分发可能再次经过过滤器，是否重复执行由注册方式和具体实现决定。

## 工作中注意

不要在通用过滤器中直接消费请求正文后假定下游还能再次读取；需要重复读取时应设计合适的包装和大小限制。日志不得记录令牌、密码或完整敏感请求体。Filter 与 Spring MVC Interceptor 所在层次不同；认证授权通常优先采用项目的安全框架和现有过滤链。

## 动手与自查

1. 验证响应头和正文，再让接口抛异常，观察结束日志是否仍出现。
2. 暂时不调用 `chain.doFilter` 并自行设置状态码，确认目标接口没有执行。
3. 两个过滤器 A、B 依次进入时，B 完成后控制流先回到哪里？

来源：[教程：使用Filter](https://liaoxuefeng.com/books/java/web/filter/index.html)、[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
