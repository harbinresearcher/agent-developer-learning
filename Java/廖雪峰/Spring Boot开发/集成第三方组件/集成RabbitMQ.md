# 集成 RabbitMQ

> **重要程度：高**｜**手练：需要**
> **重点**：交换机、路由键和队列的关系；发送与消费；确认、重试和幂等。

## 先理解消息如何流动

下单服务可以先保存订单，再发布“订单已创建”消息；库存、通知等服务分别消费消息。发送方不必同步等待每个下游处理，但消息可能延迟、重复或处理失败，业务必须设计补偿与幂等。

`生产者 → 交换机 Exchange → 绑定规则 Binding → 队列 Queue → 消费者`

- **交换机**按类型与绑定规则路由消息，不是消费者直接读取的队列。
- **队列**保存等待消费的消息；同一队列多个消费者通常竞争消息，不是每人都收到一份。
- **绑定**把交换机与队列关联；**路由键**是消息携带的路由条件。
- 默认交换机名称是空字符串，每个队列会按自身队列名自动绑定；发送时路由键填队列名即可直达该队列。
- 常见交换机：direct 按路由键精确匹配；topic 按点分词匹配（`*` 匹配一个词，`#` 匹配零个或多个词）；fanout 忽略路由键，向所有绑定队列分发。

## 动手：发送并接收一条消息

下面是一次短生命周期练习：创建随机名称队列，经默认交换机发送，然后同步拉取，最后删除本次创建的队列。它用于看清消息流向，不展示生产者确认或消费者失败重试。

准备 JDK 17、Maven 和独立的 RabbitMQ 实例。示例连接本机 `127.0.0.1:15673`，需要把测试实例的 AMQP 端口映射到该端口；确认地址指向自己的测试实例。示例使用默认 `guest` 用户，仅供本机练习。项目目录如下：

```text
rabbit-demo/
├── pom.xml
└── src/main/java/learn/rabbit/RabbitDemo.java
```

`pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>rabbit-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-amqp</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

`src/main/java/learn/rabbit/RabbitDemo.java`：

```java
package learn.rabbit;
import java.util.UUID;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
@SpringBootApplication
public class RabbitDemo {
    private static final Logger logger = LoggerFactory.getLogger(RabbitDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：发送并接收测试消息
         * 数据源：显式配置的隔离 RabbitMQ 实例
         * 操作：1) 创建唯一临时队列 2) 发送、接收并清理
         * ======================================================================== */
        logger.info("开始 RabbitMQ 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(RabbitDemo.class, args)) {
            AmqpAdmin admin = context.getBean(AmqpAdmin.class);
            RabbitTemplate rabbit = context.getBean(RabbitTemplate.class);
            // 1.1 使用唯一队列，避免与其他练习冲突
            String queue = "learn.demo." + UUID.randomUUID();
            admin.declareQueue(new Queue(queue, false, false, true));
            try {
                // 1.2 默认交换机按队列名路由，限时接收
                rabbit.convertAndSend("", queue, "hello");
                Object received = rabbit.receiveAndConvert(queue, 3000);
                System.out.println("收到=" + received);
            } finally {
                // 1.3 只删除本次创建的测试队列
                admin.deleteQueue(queue);
            }
        }
        logger.info("RabbitMQ 示例完成");
    }
}
```

在项目根目录运行：

```powershell
mvn package
java -jar target/rabbit-demo-1.0.jar --spring.rabbitmq.host=127.0.0.1 --spring.rabbitmq.port=15673
```

实际 jar 名称以 `target` 目录为准。收到消息时，输出中应包含 `收到=hello`。如果输出 `收到=null`，表示限时内没有取到消息；如果无法连接，先检查测试服务、端口和账号。

**验证边界**：此示例已用 JDK 17 和 Maven 编译；当前环境中的 RabbitMQ 实例未成功启动，因此尚未验证真实收发或上面的预期输出。

`new Queue(queue, false, false, true)` 的三个布尔值依次表示非持久化、非独占、自动删除。自动删除不等于服务重启仍保留；程序的 `finally` 只清理本次生成的队列。`receiveAndConvert` 返回 `Object`，练习中发送字符串所以可以观察到字符串；真实业务应约定消息格式和版本。

## 可靠投递：分别看两段链路

1. **生产者到 RabbitMQ**：publisher confirm 表示 broker 对发布的确认；它不表示消费者已处理业务。发布不可路由时还应配置 mandatory 和返回机制，不能只看 confirm。
2. **RabbitMQ 到消费者**：消费者处理成功后确认；未确认的投递在连接或 channel 关闭后可能重新入队。明确重试次数、退避和死信策略，避免错误消息无限循环。
3. **消费者的业务处理**：即使确认机制配置正确，网络故障或消费者崩溃仍可能重复投递。为每条业务消息设置稳定 ID，用数据库唯一约束等手段保证同一业务操作不重复生效；先处理业务再确认仍需考虑两者之间崩溃。
4. **持久化**：队列持久化与消息持久化是不同设置；还要结合队列类型、确认和故障模型判断可靠性。单独打开某个选项不等于消息绝不丢失。
5. **数据库与发消息的一致性**：数据库事务不会自动覆盖 broker。需要可靠同步时，可考虑事务消息表（outbox）、后台重发和消费者幂等，并监控积压与死信。

生产环境使用独立账号、最小权限、密钥管理和 TLS；限制消费并发与预取数量，设置连接与处理超时，监控积压、失败率和重试。不要在笔记或代码里写真实密码。

## 自测

1. 两个消费者绑定到同一个队列时，和两个队列分别绑定同一个 fanout 交换机时，消息分发有何不同？
2. publisher confirm 成功后，为什么仍不能断言业务处理成功？
3. 消费者完成数据库写入后、确认前崩溃，重投后如何避免重复扣库存？
4. 修改示例的路由键，让它与队列名不同，观察接收结果；再恢复并解释原因。

来源：[原教程](https://liaoxuefeng.com/books/java/springboot/integration/rabbitmq/index.html)；[Spring Boot AMQP](https://docs.spring.io/spring-boot/reference/messaging/amqp.html)；[RabbitMQ 确认机制](https://www.rabbitmq.com/docs/confirms)。
