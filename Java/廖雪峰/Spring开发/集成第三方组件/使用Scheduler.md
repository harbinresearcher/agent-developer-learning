# 使用 Scheduler

> **重要程度：高**｜**手练：需要**
> **重点**：`fixedDelay`、`fixedRate`、cron 的区别；调度和任务执行的区别；多实例重复执行与幂等。

## 什么时候用定时任务

定期汇总报表、清理过期记录、扫描待补偿订单都需要按时间触发。调度器决定什么时候调用任务；任务里的业务代码决定读取哪些数据、修改哪些状态。任务被触发不等于业务一定成功。

Spring 用 `@EnableScheduling` 开启注解扫描，在受 Spring 管理的 Bean 方法上写 `@Scheduled`。实例方法通常无参数；返回值不会作为业务结果交给调用方。

## 手练：每次结束后等待再运行

这个独立例子每次任务结束后等待 250 毫秒；主线程最多等 5 秒，看到两次触发后关闭容器。它只更新进程内计数器，不访问外部服务。

目录：`scheduler-demo/pom.xml` 和 `scheduler-demo/src/main/java/learn/scheduler/SchedulerDemo.java`。

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
    <groupId>learn.example</groupId><artifactId>scheduler-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

`src/main/java/learn/scheduler/SchedulerDemo.java`：

```java
package learn.scheduler;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@SpringBootApplication
@EnableScheduling
public class SchedulerDemo {
    private static final Logger logger = LoggerFactory.getLogger(SchedulerDemo.class);
    public static void main(String[] args) throws InterruptedException {
        /* ========================================================================
         * 步骤1：等待两次调度并关闭容器
         * 数据源：容器内定时任务的计数器
         * 操作：1) 启动容器 2) 有限等待后关闭
         * ======================================================================== */
        logger.info("开始调度演示");
        try (ConfigurableApplicationContext context = SpringApplication.run(SchedulerDemo.class, args)) {
            // 1.1 最多等待五秒，避免示例无限运行
            boolean completed = context.getBean(ReportJob.class).done.await(5, TimeUnit.SECONDS);
            System.out.println("两次任务=" + completed);
        }
        logger.info("调度演示结束");
    }
}
@Component
class ReportJob {
    private static final Logger logger = LoggerFactory.getLogger(ReportJob.class);
    final CountDownLatch done = new CountDownLatch(2);
    private final AtomicInteger count = new AtomicInteger();
    @Scheduled(initialDelay = 100, fixedDelay = 250)
    public void run() {
        /* ========================================================================
         * 步骤1：记录一次定时触发
         * 数据源：本进程的任务计数器
         * 操作：1) 增加计数 2) 通知等待方
         * ======================================================================== */
        logger.info("开始执行任务");
        // 1.1 更新本进程计数并通知等待方
        System.out.println("触发=" + count.incrementAndGet());
        done.countDown();
        logger.info("任务执行结束");
    }
}
```

在项目根目录运行：

```powershell
mvn package
java -jar target/scheduler-demo-1.0.jar
```

运行结果会包含 `触发=1`、`触发=2` 和 `两次任务=true`；日志中的时间和线程名会变化。本例已在 JDK 17、Spring Boot 4.1.2 下编译运行。

## 三种常用触发方式

- `fixedDelay = 250`：一次调用结束后等待 250 毫秒再安排下一次。任务用时 100 毫秒时，连续两次开始时间大约相隔 350 毫秒，实际会受调度延迟影响。
- `fixedRate = 1000`：按固定周期安排触发，以开始时间间隔为目标，不表示任务完成时间被限制为 1 秒。任务过慢、线程池设置、多个调度声明或自行把工作提交到别的线程，都可能影响重叠与延迟；不要靠假设替代并发控制。
- `cron = "0 0 2 * * *", zone = "Asia/Shanghai"`：按时区每天 02:00 触发。Spring cron 为 6 段：秒、分、时、日、月、星期；不要把带年份的其他 cron 格式照搬。时区和夏令时会影响具体触发时刻。

`initialDelay` 控制第一次触发前的等待。上例单位默认为毫秒，具体版本也支持时间单位配置。参数化时可用外部配置，变更前应验证表达式。

## 工作中最容易遗漏的边界

1. **多实例**：三份应用各有自己的调度器，同一时刻可能各跑一次。要全局只处理一次，需明确采用外部调度器、可靠的分布式协调，或数据库唯一约束与任务认领；仅靠 Java 锁只能协调同一进程。
2. **幂等**：即使有锁，进程崩溃、锁过期和重试仍可能导致重复。按业务日期和任务类型设计唯一键，用数据库约束或状态流转防止重复生效。
3. **重启与漏跑**：应用不在线时，普通进程内调度不会自动补齐所有错过的任务。保存处理进度，设计补偿扫描和恢复逻辑。
4. **任务过慢**：限制每次处理量与运行时间，设置依赖调用超时；监控上次成功时间、持续时间、失败次数与积压。
5. **错误处理**：记录任务标识、业务日期和失败原因，针对可重试错误使用有限重试；告警和人工处理仍要有入口。不要把异常吞掉并标记成功。

## 自测

1. 一次任务需要 3 秒，`fixedDelay = 1000` 时，下一次开始前至少等待哪个时间段？
2. 两个应用实例同时启动，这个例子为什么可能打印两套计数？
3. 任务完成数据库更新后、记录成功状态前崩溃，重试如何避免重复更新？

来源：[原教程](https://liaoxuefeng.com/books/java/spring/integration/scheduler/index.html)；[Spring 调度参考](https://docs.spring.io/spring-framework/reference/integration/scheduling.html)。
