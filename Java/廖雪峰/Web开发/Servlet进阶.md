# Servlet进阶

> **重要程度：高**｜**手练：需要**
> **重点**：请求流转、转发与重定向、请求作用域、会话和过滤链。

## 先把请求链串起来

客户端发送 HTTP 请求，Servlet 容器匹配路径并依次调用适用的 Filter 与目标 Servlet。Servlet 可以直接生成响应，也可以在服务端转发给其他资源，或返回重定向让客户端再次请求。请求结束后，客户端收到状态码、响应头与正文。

请求属性、会话属性和应用级共享数据的生命周期不同：请求属性只属于当前请求处理过程；Session 可关联同一会话的多个请求；应用级对象可能被许多请求共享。不要把每个用户的数据放进无隔离的 Servlet 实例字段。

## 完整例子：观察请求流转

使用 JDK 17、Maven 和 Spring Boot 4.1.2。先确认 `java -version` 与 `mvn -v` 均使用 JDK 17 或以上。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>servlet-flow</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/advanced/FlowDemo.java`：

```java
package learn.advanced;
import java.io.IOException;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
@SpringBootApplication
public class FlowDemo {
    private static final Logger logger = LoggerFactory.getLogger(FlowDemo.class);
    static class FlowServlet extends HttpServlet {
        @Override protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
            /* ========================================================================
             * 步骤1：根据路径处理请求
             * 数据源：当前 Servlet 路径
             * 操作：1) 演示转发或重定向 2) 在目标路径输出结果
             * ======================================================================== */
            logger.info("开始处理请求流转");
            try {
                // 1.1 内部转发保留请求属性，重定向由客户端再次请求
                String path = request.getServletPath();
                if ("/forward".equals(path)) {
                    request.setAttribute("marker", "转发保留");
                    request.getRequestDispatcher("/target").forward(request, response);
                    return;
                }
                if ("/redirect".equals(path)) {
                    request.setAttribute("marker", "重定向不会保留");
                    response.sendRedirect(request.getContextPath() + "/target");
                    return;
                }
                // 1.2 输出目标处理时看到的请求属性
                response.setContentType("text/plain;charset=UTF-8");
                Object marker = request.getAttribute("marker");
                response.getWriter().write("属性=" + (marker == null ? "无" : marker));
            } finally {
                logger.info("请求流转处理结束");
            }
        }
    }
    @Bean ServletRegistrationBean<FlowServlet> flowServlet() {
        return new ServletRegistrationBean<>(new FlowServlet(), "/forward", "/redirect", "/target");
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动请求流转示例
         * 数据源：Servlet 注册信息
         * 操作：1) 启动容器 2) 提供本地测试路径
         * ======================================================================== */
        logger.info("开始启动流转示例");
        // 1.1 启动内嵌 Web 服务
        SpringApplication.run(FlowDemo.class, args);
        logger.info("流转示例启动完成");
    }
}
```

在工程目录运行：

```powershell
mvn package
java -jar target/servlet-flow-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/forward
curl.exe -i http://127.0.0.1:8080/redirect
curl.exe -i -L http://127.0.0.1:8080/redirect
```

转发请求返回 200，正文为 `属性=转发保留`；不跟随的重定向返回 302 和 `Location`；`-L` 跟随新请求后正文为 `属性=无`。完成后在服务窗口按 `Ctrl+C`；端口占用时同步修改请求地址。

## 延伸到会话和过滤器

转发在同一次服务端请求处理内传递，重定向由客户端再发起请求。要跨请求关联会话，可由容器维护 Session，并通常使用客户端 Cookie 携带会话标识；这与请求属性不是一回事。

Filter 在进入目标之前和返回之后提供统一处理机会。多个过滤器的注册路径、顺序、异常和异步分发都影响结果；不能假设所有请求都会经过同一条链。Servlet 和 Session 可能被并发访问，共享可变状态需要有明确的并发策略。

## 工作中怎么用

响应提交后不要再假设状态码和响应头都能修改。重定向目标需限制为可信地址；会话标识应视为敏感凭据，配合 HTTPS、合适的 Cookie 属性、失效和 CSRF 防护。具体行为以所用容器和项目配置为准。

## 动手与自查

1. 解释三条请求的状态码和请求属性差异。
2. 用浏览器 Network 面板比较转发和重定向的请求数量。
3. 如果想跨请求保留用户登录状态，为什么请求 attribute 不够？

来源：[教程：Servlet进阶](https://liaoxuefeng.com/books/java/web/servlet-adv/index.html)、[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
