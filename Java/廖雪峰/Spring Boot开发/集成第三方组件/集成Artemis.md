# 集成Artemis

> **重要程度：高**｜**手练：需要**
> **重点**：内嵌代理、队列、配置和生产可靠性。

## Artemis 在这里做什么

ActiveMQ Artemis 是消息代理，Spring Boot 的 Artemis starter 提供客户端与 JMS 集成。本例额外加入内嵌代理依赖，使用进程内队列完成一次发送和同步接收；这是隔离练习，不是连接真实远程代理。

`spring.artemis.mode=embedded` 明确选择内嵌模式；`persistent=false` 让本例不保存代理数据；`queues` 声明演示队列。生产环境的代理部署、认证与持久化需单独配置。

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
    <groupId>learn.example</groupId><artifactId>artemis-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-artemis</artifactId></dependency>
        <dependency><groupId>org.apache.activemq</groupId><artifactId>artemis-jakarta-server</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/jms/ArtemisDemo.java`：

```java
package learn.jms;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.jms.core.JmsTemplate;
@SpringBootApplication
public class ArtemisDemo {
    private static final Logger logger = LoggerFactory.getLogger(ArtemisDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：发送并接收演示消息
         * 数据源：当前进程的内嵌 Artemis 测试队列
         * 操作：1) 发送文本 2) 限时接收并检查结果
         * ======================================================================== */
        logger.info("开始 JMS 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(ArtemisDemo.class, args)) {
            // 1.1 获取模板并设置接收等待上限
            JmsTemplate jms = context.getBean(JmsTemplate.class);
            jms.setReceiveTimeout(3000);
            jms.convertAndSend("demo.queue", "hello");
            // 1.2 同步接收，超时返回 null
            Object received = jms.receiveAndConvert("demo.queue");
            System.out.println("收到=" + received);
        }
        logger.info("JMS 示例完成");
    }
}
```

保存 `src/main/resources/application.properties`：

```properties
spring.artemis.mode=embedded
spring.artemis.embedded.persistent=false
spring.artemis.embedded.queues=demo.queue
```

确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/artemis-demo-1.0.jar
```

日志中应看到 `收到=hello`。如果出现 `JMS 示例完成` 后进程仍未退出，按 `Ctrl+C` 结束独立练习；不同版本的客户端资源关闭行为可能不同。非持久化配置也不等于代理绝不会创建分页等临时文件。

## 看懂结果与限制

应用取得 `JmsTemplate`，向 `demo.queue` 发送 `hello`，再限时接收。结果只证明当前内嵌测试代理中的一次收发；它不证明网络连接、代理重启恢复、集群或持久化配置正确。

## 工作中怎么用

外部代理应配置连接地址、认证、TLS、连接超时和监控。消息确认、重试、死信、积压和幂等必须按业务设计；消费者可能重复收到消息。生产应用中不要把本例非持久化配置误认为可靠消息保障。

## 动手与自查

1. 运行例子，确认收到的文本；不发送时观察 3 秒接收超时。
2. 在隔离环境设计代理重启实验，比较持久化配置的行为。
3. 消息发送成功和业务处理完成分别由什么证据证明？

来源：[教程：集成Artemis](https://liaoxuefeng.com/books/java/springboot/integration/artemis/index.html)、[Spring Boot JMS](https://docs.spring.io/spring-boot/reference/messaging/jms.html)。
