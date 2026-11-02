# JSP开发

> **重要程度：中**｜**手练：需要**
> **重点**：JSP 与 Servlet 的关系、请求属性、页面渲染和输出安全。

## JSP 是什么

JSP（Jakarta Server Pages）在服务器端生成响应内容，容器会将页面转换并编译为 Servlet。旧项目中常见 Servlet 接收请求、准备数据，再转发到 JSP 生成 HTML。新项目选择服务端模板还是前后端分离，应看项目架构，不必默认采用 JSP。

本例控制器设置固定请求属性并转发到 `WEB-INF` 下的 JSP。它只演示页面链路，不接收用户输入，也不实现登录。

## 完整例子

使用 JDK 17、Maven、Spring Boot 4.1.2 和内嵌 Tomcat。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>jsp-demo</artifactId><version>1.0</version><packaging>war</packaging>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.apache.tomcat.embed</groupId><artifactId>tomcat-embed-jasper</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/jsp/JspDemo.java`：

```java
package learn.jsp;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
@SpringBootApplication
@Controller
public class JspDemo {
    private static final Logger logger = LoggerFactory.getLogger(JspDemo.class);
    @GetMapping("/hello")
    public String hello(HttpServletRequest request) {
        /* ========================================================================
         * 步骤1：准备 JSP 页面数据
         * 数据源：固定的演示文案
         * 操作：1) 设置请求属性 2) 转发到 JSP
         * ======================================================================== */
        logger.info("开始准备 JSP 页面");
        // 1.1 使用可信的固定文案演示视图数据
        request.setAttribute("message", "Hello JSP");
        logger.info("JSP 页面数据准备完成");
        return "forward:/WEB-INF/jsp/hello.jsp";
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动 JSP 示例
         * 数据源：应用配置和打包的页面
         * 操作：1) 启动容器 2) 注册本地请求路径
         * ======================================================================== */
        logger.info("开始启动 JSP 示例");
        // 1.1 启动内嵌 Web 服务
        SpringApplication.run(JspDemo.class, args);
        logger.info("JSP 示例启动完成");
    }
}
```

保存 `src/main/webapp/WEB-INF/jsp/hello.jsp`：

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!doctype html>
<html lang="zh"><head><meta charset="UTF-8"><title>JSP 示例</title></head>
<body><p>${message}</p></body></html>
```

确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/jsp-demo-1.0.war --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell 请求 `curl.exe -i http://127.0.0.1:8080/hello`。应收到 HTTP 200 和包含 `Hello JSP` 的 HTML。结束后在服务窗口按 `Ctrl+C`；端口被占用时同步修改请求地址。

## 看懂页面处理

控制器把 `message` 放入请求属性，返回转发路径；容器将同一请求交给 JSP，表达式 `${message}` 读取属性并输出。`WEB-INF` 下的资源通常不直接作为公开 URL 访问，而由服务器内部转发。

本例内容是固定可信字符串。JSP 表达式输出不应被视为自动完成所有 HTML 转义；展示不可信数据时要按上下文使用合适的转义与模板机制，避免跨站脚本。不要在 JSP 混入大量数据库访问或业务逻辑。

## 工作中注意

本例用可执行 WAR 和 Jasper；Spring Boot 的可执行 JAR 对 JSP 有已知限制，不能把一般 JAR 示例改名就假定 JSP 可用。外部容器部署还需核对 Jakarta Servlet、JSP 及应用版本兼容；本例仅验证本机可执行 WAR。

## 动手与自查

1. 请求 `/hello` 并查看页面源码；再尝试直接访问 `WEB-INF` 下的 JSP。
2. 修改固定消息并重新构建，核对渲染结果。
3. 为什么不能把用户输入直接输出到 HTML 而不考虑转义？

来源：[教程：JSP开发](https://liaoxuefeng.com/books/java/web/jsp/index.html)、[Spring Boot JSP 限制](https://docs.spring.io/spring-boot/reference/web/servlet.html)。
