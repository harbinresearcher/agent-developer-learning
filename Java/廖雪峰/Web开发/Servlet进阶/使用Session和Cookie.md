# 使用Session和Cookie

> **重要程度：高**｜**手练：需要**
> **重点**：会话标识、Cookie 往返、失效、并发与安全。

## 先分清两种东西

Cookie 是服务器通过响应头让客户端保存的一段数据；客户端后续按 Cookie 的作用域和规则携带它。Session 是服务端维护的会话状态；容器通常用会话标识把不同请求关联到对应 Session。本例用默认 Cookie 跟踪会话，具体跟踪方式取决于容器配置。

客户端第一次 POST 时没有会话标识，容器创建 Session 并返回会话 Cookie；客户端下一次携带它，服务端读到原来的计数。换一个没有 Cookie 的客户端会得到另一个会话。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>servlet-session</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/advanced/SessionDemo.java`：

```java
package learn.advanced;
import java.io.IOException;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
@SpringBootApplication
public class SessionDemo {
    private static final Logger logger = LoggerFactory.getLogger(SessionDemo.class);
    static class CounterServlet extends HttpServlet {
        @Override protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
            /* ========================================================================
             * 步骤1：更新会话计数
             * 数据源：当前客户端对应的测试会话
             * 操作：1) 创建或读取会话 2) 更新并返回计数
             * ======================================================================== */
            logger.info("开始更新会话计数");
            try {
                // 1.1 获取会话并读取此前值
                HttpSession session = request.getSession(true);
                Integer previous = (Integer) session.getAttribute("count");
                int count = previous == null ? 1 : previous + 1;
                // 1.2 保存当前值，不记录会话标识
                session.setAttribute("count", count);
                response.setContentType("text/plain;charset=UTF-8");
                response.getWriter().write("计数=" + count);
            } finally {
                logger.info("会话计数更新结束");
            }
        }
    }
    @Bean ServletRegistrationBean<CounterServlet> counterServlet() {
        return new ServletRegistrationBean<>(new CounterServlet(), "/counter");
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动会话示例
         * 数据源：内嵌 Servlet 容器
         * 操作：1) 启动应用 2) 注册计数路径
         * ======================================================================== */
        logger.info("开始启动会话示例");
        // 1.1 启动本地 Web 服务
        SpringApplication.run(SessionDemo.class, args);
        logger.info("会话示例启动完成");
    }
}
```

## 本机验证

确认 `java -version` 和 `mvn -v` 都使用 JDK 17 或以上。在工程目录运行：

```powershell
mvn package
java -jar target/servlet-session-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell，在当前工作目录保存演示 Cookie 后再发送：

```powershell
curl.exe -i -X POST -c session-demo.txt http://127.0.0.1:8080/counter
curl.exe -i -X POST -b session-demo.txt -c session-demo.txt http://127.0.0.1:8080/counter
curl.exe -i -X POST http://127.0.0.1:8080/counter
```

正文依次为 `计数=1`、`计数=2`、`计数=1`。第一条响应通常包含会话 `Set-Cookie`，第二条用文件中的 Cookie 关联同一会话；第三条未带 Cookie。文件仅用于本地练习，不要复制真实登录会话。完成后在服务窗口按 `Ctrl+C`；端口占用时同步修改启动和请求地址。

## 生命周期和安全边界

`request.getSession(true)` 在需要时创建会话；`getSession(false)` 在不存在时返回 null。退出登录时应按应用流程使会话失效并清理相关状态；会话过期时间和存储方式由容器或项目配置决定。

会话标识相当于敏感凭据，不应写入日志或放在公开链接。生产环境应结合 HTTPS 和需求设置 Cookie 的 `Secure`、`HttpOnly`、`SameSite`、作用路径与过期策略，并考虑登录后的会话固定攻击和 CSRF；属性本身不替代完整的认证与授权。

本例的读取、加一、写回不是原子操作，同一会话并发请求可能丢失更新。生产系统应按实际存储和并发模型设计原子更新；多个应用实例也不能假设内存 Session 自动共享。

## 动手与自查

1. 连续发送两次带同一 Cookie 的 POST，再不带 Cookie 发送，比较计数。
2. 在测试中调用 `session.invalidate()`，观察旧会话是否还能被使用。
3. 为什么 `SameSite` 有助于限制部分跨站携带 Cookie 的情况，但不应单独当作完整 CSRF 方案？

来源：[教程：使用Session和Cookie](https://liaoxuefeng.com/books/java/web/servlet-adv/session-cookie/index.html)、[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
