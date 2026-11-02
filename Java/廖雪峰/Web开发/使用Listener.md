# 使用Listener

> **重要程度：中**｜**手练：需要**
> **重点**：容器生命周期事件、注册方式、会话创建与销毁。

## Listener 监听什么

Servlet Listener 接收容器产生的生命周期事件，例如应用上下文启动、会话创建或销毁。它与 Filter 不同：Filter 位于请求处理链，Listener 响应对应事件；并非每个请求都会触发一次会话创建事件。

本例用 `HttpSessionListener` 观察会话创建和销毁，用 Spring Boot 注册监听器；接口仅用于本地测试，不代表完整登录系统。

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
    <groupId>learn.example</groupId><artifactId>listener-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/listener/ListenerDemo.java`：

```java
package learn.listener;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpSession;
import jakarta.servlet.http.HttpSessionEvent;
import jakarta.servlet.http.HttpSessionListener;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletListenerRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
@SpringBootApplication
@RestController
public class ListenerDemo {
    private static final Logger logger = LoggerFactory.getLogger(ListenerDemo.class);
    @Bean ServletListenerRegistrationBean<HttpSessionListener> sessionListener() {
        HttpSessionListener listener = new HttpSessionListener() {
            @Override public void sessionCreated(HttpSessionEvent event) {
                /* ========================================================================
                 * 步骤1：记录会话创建
                 * 数据源：容器的会话生命周期事件
                 * 操作：1) 接收创建事件 2) 写入非敏感日志
                 * ======================================================================== */
                logger.info("开始处理会话创建事件");
                // 1.1 不输出会话标识或用户信息
                logger.info("会话创建事件处理完成");
            }
            @Override public void sessionDestroyed(HttpSessionEvent event) {
                /* ========================================================================
                 * 步骤1：记录会话销毁
                 * 数据源：容器的会话生命周期事件
                 * 操作：1) 接收销毁事件 2) 写入非敏感日志
                 * ======================================================================== */
                logger.info("开始处理会话销毁事件");
                // 1.1 不输出会话标识或用户信息
                logger.info("会话销毁事件处理完成");
            }
        };
        return new ServletListenerRegistrationBean<>(listener);
    }
    @PostMapping("/session/start") public String start(HttpServletRequest request) {
        /* ========================================================================
         * 步骤1：创建会话
         * 数据源：当前请求
         * 操作：1) 获取或创建会话 2) 返回结果
         * ======================================================================== */
        logger.info("开始创建会话");
        // 1.1 创建或获取当前会话
        request.getSession(true);
        logger.info("创建会话结束");
        return "created";
    }
    @PostMapping("/session/end") public String end(HttpServletRequest request) {
        /* ========================================================================
         * 步骤1：结束会话
         * 数据源：当前请求的已有会话
         * 操作：1) 查找会话 2) 存在时使其失效
         * ======================================================================== */
        logger.info("开始结束会话");
        try {
            // 1.1 不创建新会话
            HttpSession session = request.getSession(false);
            if (session == null) return "absent";
            // 1.2 显式使会话失效
            session.invalidate();
            return "ended";
        } finally {
            logger.info("结束会话操作完成");
        }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动监听器示例
         * 数据源：容器与已注册的监听器
         * 操作：1) 启动服务 2) 提供本地演示接口
         * ======================================================================== */
        logger.info("开始启动监听器示例");
        // 1.1 启动内嵌 Web 服务
        SpringApplication.run(ListenerDemo.class, args);
        logger.info("监听器示例启动完成");
    }
}
```

确认 `java -version` 与 `mvn -v` 均使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/listener-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell，保存并重用演示 Cookie：

```powershell
curl.exe -X POST -c listener-cookie.txt http://127.0.0.1:8080/session/start
curl.exe -X POST -b listener-cookie.txt http://127.0.0.1:8080/session/end
curl.exe -X POST -b listener-cookie.txt http://127.0.0.1:8080/session/end
```

响应依次为 `created`、`ended`、`absent`；服务日志分别出现会话创建和销毁事件。完成后在服务窗口按 `Ctrl+C`。这个 Cookie 文件仅供本机演示，不应存放真实登录凭据。

## 工作边界

`getSession(true)` 可以创建会话，`getSession(false)` 不会；`invalidate()` 使当前会话失效并触发相应生命周期处理。会话也可能因超时销毁，具体时机由容器管理，不应把回调时间当作精确计时器。

监听器应快速、可靠地处理事件；不要在回调中执行无界耗时任务，或把会话标识、密码写入日志。多个实例和异常退出会影响生命周期事件的观察，不能单靠监听器保证分布式业务操作恰好执行一次。

## 动手与自查

1. 连续执行三条请求，核对响应和创建、销毁日志。
2. 在不带 Cookie 的新客户端直接调用结束接口，解释为什么不会创建会话。
3. Filter 和 Session Listener 各在什么时机运行？

来源：[教程：使用Listener](https://liaoxuefeng.com/books/java/web/listener/index.html)、[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
