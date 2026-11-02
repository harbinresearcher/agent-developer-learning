# 集成JMS

> **重要程度：高**｜**手练：需要**
> **重点**：JMS 抽象、队列、发送接收和可靠性边界。

## JMS 是什么

JMS 是 Java 消息服务的 API 规范，定义连接、消息、目的地等抽象；Artemis 是可以提供 JMS 支持的消息代理。Spring 的 `JmsTemplate` 封装常见发送和同步接收操作。队列通常用于竞争消费，主题通常用于发布订阅；实际持久化、确认和订阅语义需结合代理配置。

例子启动当前进程内的非持久化 Artemis 测试代理，发送一条文本并限时接收，不连接外部消息服务。

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

## 看懂收发

`convertAndSend` 将字符串转换为 JMS 消息并发送到 `demo.queue`；`receiveAndConvert` 等待最多 3 秒，超时可能返回 null。例子只验证单进程中的一次同步收发，进程退出后内存中的消息不会保留。

## 工作中怎么用

必须明确确认方式、事务、持久化、重试和死信策略；消息可能重复投递，消费者应根据业务实现幂等。数据库事务与消息提交不是天然的一个原子操作，需要按系统设计事务消息或其他可靠方案。

连接地址、认证、TLS 和超时应按环境配置，避免记录敏感消息。现代 Spring 使用 Jakarta JMS；旧项目的 `javax.jms` 与 `jakarta.jms` 版本需要核对。

## 动手与自查

1. 运行示例并检查接收结果；去掉发送步骤，观察接收超时。
2. 解释队列和主题各适合什么消费关系。
3. 为什么一次本地收发成功不能证明消息在进程异常后仍然存在？

来源：[教程：集成JMS](https://liaoxuefeng.com/books/java/spring/integration/jms/index.html)、[Spring Boot JMS](https://docs.spring.io/spring-boot/reference/messaging/jms.html)。
