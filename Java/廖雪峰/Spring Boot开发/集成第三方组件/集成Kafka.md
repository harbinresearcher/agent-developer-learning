# 集成Kafka

> **重要程度：高**｜**手练：需要**
> **重点**：主题、分区、生产者、消费者组、偏移量和可靠性。

## Kafka 的基本概念

生产者把记录写入主题，主题由一个或多个分区组成；消费者通过消费者组读取记录并跟踪偏移量。同一分区内有顺序，不能由此推断跨分区的全局顺序。记录的键可影响分区选择，具体取决于分区器和配置。

本例用 Spring Kafka 的内嵌 KRaft 测试代理验证一条消息，适合本机学习；它不依赖外部 Kafka 服务，也不能代表生产集群的可靠性。

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
    <groupId>learn.example</groupId><artifactId>kafka-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-kafka</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-kafka-test</artifactId><scope>test</scope></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-test</artifactId><scope>test</scope></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/kafka/KafkaDemo.java`：

```java
package learn.kafka;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class KafkaDemo { }
```

保存 `src/test/java/learn/kafka/KafkaFlowTest.java`：

```java
package learn.kafka;
import java.time.Duration;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.test.EmbeddedKafkaBroker;
import org.springframework.kafka.test.context.EmbeddedKafka;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import static org.junit.jupiter.api.Assertions.assertEquals;
@SpringBootTest(properties = {
    "spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer",
    "spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer"
})
@EmbeddedKafka(partitions = 1, topics = "demo.events")
class KafkaFlowTest {
    private static final Logger logger = LoggerFactory.getLogger(KafkaFlowTest.class);
    @Autowired KafkaTemplate<String, String> producer;
    @Autowired EmbeddedKafkaBroker broker;
    @Test void sendsAndReadsOneRecord() throws Exception {
        /* ========================================================================
         * 步骤1：验证消息发送和读取
         * 数据源：隔离的内嵌 Kafka 测试代理
         * 操作：1) 创建消费者并订阅 2) 发送并验证记录
         * ======================================================================== */
        logger.info("开始 Kafka 收发测试");
        // 1.1 为测试组创建字符串消费者
        Map<String, Object> properties = KafkaTestUtils.consumerProps("demo-group", "false", broker);
        DefaultKafkaConsumerFactory<String, String> factory = new DefaultKafkaConsumerFactory<>(
            properties, new StringDeserializer(), new StringDeserializer());
        try (Consumer<String, String> consumer = factory.createConsumer()) {
            broker.consumeFromAnEmbeddedTopic(consumer, "demo.events");
            // 1.2 等待发送完成并读取目标主题记录
            producer.send("demo.events", "order-1", "created").get(10, TimeUnit.SECONDS);
            ConsumerRecord<String, String> record = KafkaTestUtils.getSingleRecord(consumer, "demo.events", Duration.ofSeconds(10));
            assertEquals("order-1", record.key());
            assertEquals("created", record.value());
        }
        logger.info("Kafka 收发测试完成");
    }
}
```

确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn test
```

测试启动隔离代理，发送键为 `order-1`、值为 `created` 的记录，再读取并断言键和值。应检查测试报告中的实际通过数量，而不是只看应用日志。

## 工作中需要明确什么

发送成功的含义受生产者确认、重试和代理配置影响；消费者何时提交偏移量决定失败后可能重读或漏处理的边界。重复投递和处理失败需要结合业务实现幂等、重试与死信策略，不能仅靠消费者组名称保证恰好一次业务效果。

生产环境需要明确分区数、副本、认证、TLS、保留策略和监控；不同版本配置应查对应文档。内嵌测试不证明跨机器网络、权限、故障恢复或集群高可用。

## 动手与自查

1. 运行测试，核对报告中的测试数和结果。
2. 修改消息值和断言，观察一致与不一致时的结果。
3. 为什么同一消费者组重启后可能再次处理某些记录？

来源：[教程：集成Kafka](https://liaoxuefeng.com/books/java/springboot/integration/kafka/index.html)、[Spring Kafka 测试](https://docs.spring.io/spring-kafka/reference/testing.html)、[Spring Boot Kafka](https://docs.spring.io/spring-boot/reference/messaging/kafka.html)。
