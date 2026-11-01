# Servlet开发

> **重要程度：高**｜**手练：需要**
> **重点**：请求参数、状态码、字符编码、响应提交和异常路径。

## 请求和响应对象

`HttpServletRequest` 描述客户端请求；`getParameter("name")` 可读取本例 URL 查询参数。`HttpServletResponse` 用于设置状态码、响应头与正文。示例缺失、空白或超过 40 个字符时返回 400；合法输入返回 UTF-8 文本。

该例只展示 GET 与查询参数，不假设所有请求参数都来自 JSON。JSON 正文通常需要读取请求体并按约定解析；同一个请求体的读取方式和重复读取限制应按容器与框架契约处理。

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

## 看懂示例和边界

`name` 从请求读取到局部变量，先检查是否为 null，再检查空白与长度。`sendError(400, ...)` 后立即返回，避免继续写成功内容。合法路径先设置 `Content-Type` 与字符编码，再获取 writer 写正文；HTTP 响应一旦提交，状态码与头部通常就不能再随意修改。

示例用 `text/plain`，没有把输入放进 HTML。若在 HTML、SQL、URL 或日志中使用输入，必须分别按目标上下文处理，不能认为长度检查自动防止所有注入。日志避免记录密码和令牌。

`HttpServlet` 默认会按 HTTP 方法分派；本例没有实现 POST，实测 POST 返回 405。实际项目应明确参数缺失、数据不合法、服务异常和响应格式，不把所有情况都当成 200。

## 动手与自查

1. 传入合法名字、缺失参数和空字符串，比较正文与状态码。
2. 实现 `doPost`，先明确请求正文格式和错误响应，再用客户端验证。
3. 为什么应该先设置响应类型和编码，再获取 writer 并写正文？

来源：[教程：Servlet开发](https://liaoxuefeng.com/books/java/web/servlet-dev/index.html)、[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
