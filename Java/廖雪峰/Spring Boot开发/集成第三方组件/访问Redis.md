# 访问Redis

> **重要程度：高**｜**手练：需要**
> **重点**：StringRedisTemplate、过期时间、连接配置、缓存一致性与失败处理。

## Redis 在应用中做什么

Redis 是内存数据存储，常用于缓存、计数或其他低延迟数据操作。Spring Boot 可根据配置创建连接工厂和 `StringRedisTemplate`；后者便于读写字符串键和值。Redis 并不自动替代数据库事务或持久化方案。

本例连接隔离测试实例，生成唯一键，写入带过期时间的值、读取并删除该键。不要指向生产实例。

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
    <groupId>learn.example</groupId><artifactId>redis-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-data-redis</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/redis/RedisDemo.java`：

```java
package learn.redis;
import java.time.Duration;
import java.util.UUID;
import java.util.concurrent.TimeUnit;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.data.redis.core.StringRedisTemplate;
@SpringBootApplication
public class RedisDemo {
    private static final Logger logger = LoggerFactory.getLogger(RedisDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：写入并读取测试缓存
         * 数据源：显式配置的隔离 Redis 实例
         * 操作：1) 写入带过期时间的唯一键 2) 查询并清理该键
         * ======================================================================== */
        logger.info("开始 Redis 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(RedisDemo.class, args)) {
            StringRedisTemplate redis = context.getBean(StringRedisTemplate.class);
            // 1.1 使用唯一键，避免与其他练习相互覆盖
            String key = "learn:demo:" + UUID.randomUUID();
            try {
                redis.opsForValue().set(key, "Alice", Duration.ofSeconds(60));
                // 1.2 读取值并检查过期时间
                System.out.println("值=" + redis.opsForValue().get(key));
                Long ttl = redis.getExpire(key, TimeUnit.SECONDS);
                System.out.println("TTL有效=" + (ttl != null && ttl > 0));
            } finally {
                // 1.3 只删除本次生成的测试键
                redis.delete(key);
            }
        }
        logger.info("Redis 示例完成");
    }
}
```

先准备一个仅供测试的 Redis 实例，并确认它监听 `127.0.0.1:16379`。确认 `java -version` 与 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/redis-demo-1.0.jar --spring.data.redis.host=127.0.0.1 --spring.data.redis.port=16379
```

日志中应看到：

```text
值=Alice
TTL有效=true
```

程序生成随机测试键，`set` 同时设置值和 60 秒过期时间，随后读取值与剩余过期时间，最后只删除该测试键。该布尔输出要求查询时尚有正的剩余秒数；调试停顿过久或实例不可用时结果可能不同。练习结束后按测试实例的管理方式停止它。

## 工作中怎么用

缓存键应有明确命名、数据类型与过期策略；TTL 可以限制陈旧数据存在时间，但不自动保证数据库和缓存强一致。要结合更新顺序、失败重试和业务容忍度设计一致性方案。

生产连接地址、认证和 TLS 按部署环境配置，凭据不写入源码或日志。设置连接与操作超时，处理实例不可用、连接池耗尽和降级；本例的 `finally` 清理若失败也可能抛异常，生产代码应保留原始错误并记录清理失败。

Redis 命令的原子性不等于多个命令的业务流程自动原子；并发扣减、分布式锁和重试需要各自明确边界。不要使用 `FLUSHDB` 等全库操作清理练习数据。

## 动手与自查

1. 在隔离实例运行示例，核对读取值和 TTL。
2. 调整过期时间并等待过期，再验证读取结果；注意使用独立测试键。
3. 为什么缓存设了 TTL，数据库和缓存仍可能暂时不一致？

来源：[教程：访问Redis](https://liaoxuefeng.com/books/java/springboot/integration/redis/index.html)、[Spring Boot Redis](https://docs.spring.io/spring-boot/reference/data/nosql.html)。
