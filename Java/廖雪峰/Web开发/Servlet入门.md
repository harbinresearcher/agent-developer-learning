# Servlet入门

> **重要程度：高**｜**手练：需要**
> **重点**：容器、映射、生命周期与并发请求。

## Servlet 是什么

Servlet 是运行在 Servlet 容器中的 Java Web 组件。容器负责接收请求、按映射找到目标 Servlet，并调用对应方法；Servlet 通过请求与响应对象处理一次交互。Tomcat 是常见的 Servlet 容器；本例用 Spring Boot 启动内嵌 Tomcat 来省去手动部署步骤。

本例通过 `ServletRegistrationBean` 把 `GreetingServlet` 注册到 `/greet`，继承 `HttpServlet` 并覆盖 `doGet`。Spring MVC 控制器和 Servlet 处在不同层次：MVC 请求通常由 `DispatcherServlet` 等组件处理，本例直接注册自己的 Servlet。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。先用 `java -version` 与 `mvn -v` 确认使用 JDK 17 或以上。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>servlet-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/servlet/ServletDemo.java`：

```java
package learn.servlet;
import java.io.IOException;
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
public class ServletDemo {
    private static final Logger logger = LoggerFactory.getLogger(ServletDemo.class);
    static class GreetingServlet extends HttpServlet {
        @Override protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
            /* ========================================================================
             * 步骤1：处理问候请求
             * 数据源：GET 请求的 name 查询参数
             * 操作：1) 校验参数 2) 设置响应类型并输出
             * ======================================================================== */
            logger.info("开始处理问候请求");
            try {
                // 1.1 对缺失和过长参数返回客户端错误
                String name = request.getParameter("name");
                if (name == null || name.isBlank() || name.length() > 40) {
                    response.sendError(HttpServletResponse.SC_BAD_REQUEST, "invalid name");
                    return;
                }
                // 1.2 先设置编码和类型，再写入响应正文
                response.setContentType("text/plain;charset=UTF-8");
                response.getWriter().write("你好，" + name);
            } finally {
                logger.info("问候请求处理结束");
            }
        }
    }
    @Bean ServletRegistrationBean<GreetingServlet> greetingServlet() {
        return new ServletRegistrationBean<>(new GreetingServlet(), "/greet");
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动 Servlet 示例
         * 数据源：Boot 配置与 Servlet 注册信息
         * 操作：1) 启动内嵌容器 2) 注册请求映射
         * ======================================================================== */
        logger.info("开始启动 Servlet 示例");
        // 1.1 由容器管理 Servlet 生命周期
        SpringApplication.run(ServletDemo.class, args);
        logger.info("Servlet 示例启动完成");
    }
}
```

在工程目录运行：

```powershell
mvn package
java -jar target/servlet-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell 请求：

```powershell
curl.exe -i "http://127.0.0.1:8080/greet?name=Alice"
curl.exe -i "http://127.0.0.1:8080/greet"
curl.exe -i -X POST "http://127.0.0.1:8080/greet"
```

依次应得到 200（正文 `你好，Alice`）、400、405。使用中文参数时客户端需要按 URL 编码发送；本地验证可用 `curl.exe -G --data-urlencode "name=小明" http://127.0.0.1:8080/greet`。完成后在服务窗口按 `Ctrl+C`；端口被占用时修改启动参数和请求地址。

## 调用链和生命周期

客户端发送 `GET /greet?name=...`，容器按 `/greet` 映射找到 Servlet，`HttpServlet` 分派到 `doGet`；方法写入响应正文。容器负责实例化或管理已注册实例，并在适当时机执行初始化和销毁回调。不要自行假设每个请求都会创建一个新 Servlet。

Servlet 通常会被多个请求并发调用：请求专属数据放在方法局部变量，不要放到共享的可变实例字段中。数据库连接等资源仍需按各自契约管理。

## 工作中怎么用

现代 Jakarta Servlet 使用 `jakarta.servlet` 包；旧版 Java EE 项目可能仍使用 `javax.servlet`，两者需要匹配对应容器与框架版本。Spring Boot 4 和本例使用 Jakarta 命名空间。实际业务常用 Spring MVC，但理解 Servlet、Filter 和容器有助于排查请求链。

## 动手与自查

1. 将注册路径改为 `/welcome`，验证旧路径和新路径的结果。
2. 解释为什么两个并发请求不能用 Servlet 的共享字段保存各自的 `name`。
3. Servlet 容器和业务代码分别负责什么？

来源：[教程：Servlet入门](https://liaoxuefeng.com/books/java/web/servlet-basic/index.html)、[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
