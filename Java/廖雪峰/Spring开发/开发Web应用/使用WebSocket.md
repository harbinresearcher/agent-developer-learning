# 使用WebSocket

> **重要程度：高**｜**手练：需要**
> **重点**：握手、双向消息、连接生命周期、来源与认证。

## WebSocket 解决什么问题

WebSocket 在 HTTP 握手成功后建立可双向收发消息的连接，适合聊天、实时通知等需要持续通信的场景。它不是普通 HTTP 请求自动变成可靠消息队列；断线、重连、消息顺序和重复处理仍需应用设计。

本例用 Spring `TextWebSocketHandler` 在 `/echo` 接收文本并向当前连接回复。只允许演示来源 `http://localhost:8080`；服务只绑定本机，不实现用户认证，也不广播消息。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>websocket-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-websocket</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/ws/WebSocketDemo.java`：

```java
package learn.ws;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.socket.handler.TextWebSocketHandler;
@SpringBootApplication
@EnableWebSocket
public class WebSocketDemo implements WebSocketConfigurer {
    private static final Logger logger = LoggerFactory.getLogger(WebSocketDemo.class);
    static class EchoHandler extends TextWebSocketHandler {
        @Override protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
            /* ========================================================================
             * 步骤1：处理一条文本消息
             * 数据源：当前 WebSocket 会话收到的文本
             * 操作：1) 限制文本长度 2) 向当前会话回复
             * ======================================================================== */
            logger.info("开始处理 WebSocket 消息");
            try {
                // 1.1 拒绝过长文本，不记录消息内容
                String text = message.getPayload();
                if (text.length() > 128) {
                    session.close(CloseStatus.POLICY_VIOLATION);
                    return;
                }
                // 1.2 仅向当前连接发送演示回复
                session.sendMessage(new TextMessage("收到:" + text));
            } finally {
                logger.info("WebSocket 消息处理结束");
            }
        }
    }
    @Override public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new EchoHandler(), "/echo").setAllowedOrigins("http://localhost:8080");
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动 WebSocket 示例
         * 数据源：本地服务配置和处理器
         * 操作：1) 启动容器 2) 注册连接路径
         * ======================================================================== */
        logger.info("开始启动 WebSocket 示例");
        // 1.1 启动嵌入式服务器
        SpringApplication.run(WebSocketDemo.class, args);
        logger.info("WebSocket 示例启动完成");
    }
}
```

先确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/websocket-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

在浏览器打开 `http://localhost:8080`，即使该路径显示 404，也可在该页面的开发者工具控制台运行：

```javascript
/* ========================================================================
 * 步骤1：连接并发送演示消息
 * 数据源：本机 WebSocket 接口
 * 操作：1) 建立连接并发送 2) 接收回复后关闭
 * ======================================================================== */
const logger = console;
logger.info("开始 WebSocket 练习");
// 1.1 连接当前本机服务
const socket = new WebSocket("ws://localhost:8080/echo");
socket.onopen = () => socket.send("hello");
// 1.2 显示回复并关闭连接
socket.onmessage = event => { logger.info(event.data); socket.close(); logger.info("练习完成"); };
socket.onerror = () => logger.info("连接失败，请检查服务和来源");
```

应看到 `收到:hello`。完成后在服务窗口按 `Ctrl+C`。如果更换端口，还需同步修改启动端口、浏览器地址、WebSocket 地址和允许来源中的端口。

## 连接与安全边界

握手阶段检查来源，成功后消息由处理器接收；例子对超过 128 个 Java 字符单元的文本关闭连接。该检查发生在处理器收到消息后，实际服务还应在容器和网关配置消息大小、连接数及空闲超时限制。

来源检查不是用户认证或授权，非浏览器客户端可自行设置请求头。生产环境需要认证、逐消息授权、`wss`、反向代理支持、心跳和断线重连策略；日志不要记录令牌或敏感消息。

多个执行路径向同一会话发送消息时，还需遵守底层会话的并发发送契约。是否使用 STOMP、消息代理或持久化由业务需求决定，本例没有实现这些能力。

## 动手与自查

1. 建立连接并发送文本，确认回复后主动关闭。
2. 修改允许来源或发送超长文本，观察握手或连接结果。
3. 网络断开后，为什么不能假设断线期间的消息一定会自动补发？

来源：[教程：使用WebSocket](https://liaoxuefeng.com/books/java/spring/web/websocket/index.html)、[Spring WebSocket API](https://docs.spring.io/spring-framework/reference/web/websocket/server.html)。
