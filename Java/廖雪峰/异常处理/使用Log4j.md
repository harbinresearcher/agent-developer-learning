# 使用 Log4j 2

> **重要程度：中**｜**手练：按需**
> **重点**：Log4j API 与 Core、级别和输出、配置、版本与安全维护。

## 先分清 Log4j 1 与 Log4j 2

Log4j 提供日志 API 和实现，可配置日志级别、格式和输出位置。Log4j 1 已于 2015 年停止维护；阅读旧教程或维护旧项目时，不能把 1.x 的包名和配置当作 Log4j 2。新项目应按项目的 JDK 和依赖策略选择仍受维护的版本，核对官方安全公告。

## 完整例子

下面使用 Log4j 2 API 和 Core。保存为 `Log4jDemo.java`；在 Maven 项目中把示例配置保存为 `src/main/resources/log4j2.xml`，使它进入运行类路径。

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
public class Log4jDemo {
    private static final Logger logger = LogManager.getLogger(Log4jDemo.class);
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：记录订单处理事件
         * ========================================================================
         * 数据源：演示订单编号和模拟异常
         * 操作：
         * 1) 使用占位符记录编号
         * 2) 记录异常并保留堆栈
         */
        logger.info("开始处理订单");
        // 1.1 记录安全的订单编号
        int orderId = 42;
        logger.info("订单编号: {}", orderId);
        // 1.2 记录模拟失败和异常对象
        try {
            throw new IllegalStateException("模拟失败");
        } catch (IllegalStateException ex) {
            logger.error("订单处理失败", ex);
        }
        logger.info("订单处理演示完成");
    }
}
```

`log4j2.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%level %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

Maven `<dependencies>` 内的依赖片段如下；它不是完整 `pom.xml`：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.25.5</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.25.5</version>
    <scope>runtime</scope>
</dependency>
```

示例已用 Log4j API 和 Core 2.25.5、配置文件及 Java 11 编译运行；该 Maven 片段未单独构建。日志包含 `INFO 订单编号: 42`、`ERROR 订单处理失败` 与异常堆栈。时间、堆栈行号及具体格式随配置和运行环境变化。直接用 `java` 运行时，类路径必须包含应用 class、配置文件以及 API 和 Core 依赖。

## 执行过程

`LogManager.getLogger` 获取 Log4j 2 API 的日志器。INFO 消息与 ERROR 异常被交给 Core；配置把根日志级别设为 INFO，并使用 Console appender 和 PatternLayout 输出级别与消息。`logger.error(..., ex)` 传递异常对象，便于后端记录堆栈。示例捕获后继续仅用于展示日志，并不表示订单处理成功。

API 和实现不是同一个工件；只引入 API、配置没有进运行类路径、级别被过滤，或应用使用了另一个后端，都可能使结果与预期不同。排查时先确认依赖树、真正生效的配置文件和输出端。

## 工作中怎么用

不要照搬历史版本作为长期安全基线；按项目支持的 JDK、官方维护与漏洞公告升级，并测试配置和桥接。Log4j 1 到 2 存在包名和配置差异，应按官方迁移指南处理。日志与配置不应包含密码和令牌；输入来自外部时也需遵守项目的日志处理规范。

## 手练与答案

1. 将 Root 级别设为 `error`：INFO 消息会被过滤，ERROR 消息仍可能输出，具体还受其他 logger 和过滤器配置影响。
2. 把配置文件移出运行类路径：本例配置不再生效，结果取决于 Log4j 2 的其他可用配置和默认行为。
3. 把 `ex` 从 error 调用中去掉：这条记录不再携带原异常对象，堆栈信息减少。

参考：[廖雪峰：Log4j](https://liaoxuefeng.com/books/java/exception/log4j/index.html)；[Apache：从 Log4j 1 迁移](https://logging.apache.org/log4j/2.x/migrate-from-log4j1.html)；[Apache：配置](https://logging.apache.org/log4j/2.x/manual/configuration)。
