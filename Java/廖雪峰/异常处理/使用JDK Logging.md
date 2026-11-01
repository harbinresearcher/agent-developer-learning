# 使用 JDK Logging

> **重要程度：高**｜**手练：需要**
> **重点**：日志级别、参数化消息、异常堆栈、配置与敏感信息。

## 为什么使用日志

日志记录程序运行时的重要事件和错误，便于定位问题。JDK 自带 `java.util.logging`（JUL），无需引入第三方库；它与 SLF4J、Logback、Log4j 2 的职责和配置不同，项目中应遵循已有约定。

## 完整例子

保存为 `JdkLogDemo.java`，用 `javac -encoding UTF-8 JdkLogDemo.java` 编译，再运行 `java JdkLogDemo`。默认控制台日志通常写到标准错误，时间、格式和是否显示取决于配置。

```java
import java.util.logging.Level;
import java.util.logging.Logger;
public class JdkLogDemo {
    private static final Logger logger = Logger.getLogger(JdkLogDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：记录任务结果和异常
         * ========================================================================
         * 数据源：本地订单编号与模拟故障
         * 操作：
         * 1) 记录任务开始和订单编号
         * 2) 捕获模拟异常并附带原始异常记录
         */
        logger.info("开始处理订单");
        // 1.1 以参数化形式记录安全的业务标识
        int orderId = 42;
        logger.log(Level.INFO, "订单编号: {0}", orderId);
        // 1.2 演示带异常对象的日志记录
        try {
            throw new IllegalStateException("模拟失败");
        } catch (IllegalStateException ex) {
            logger.log(Level.WARNING, "订单处理失败", ex);
        }
        logger.info("订单处理演示完成");
    }
}
```

在默认配置下，日志包含“开始处理订单”“订单编号: 42”“订单处理失败”、`IllegalStateException: 模拟失败` 及堆栈，最后包含“订单处理演示完成”。堆栈行号和时间会随环境变化。

## 执行过程

`Logger.getLogger(...)` 取得以类名命名的日志器。`logger.info` 记录普通事件；`logger.log(Level.INFO, "订单编号: {0}", orderId)` 使用 JUL 的参数占位符 `{0}`，不是 SLF4J 的 `{}`。模拟操作抛出异常后，catch 使用接收 `Throwable` 的重载记录警告与原始堆栈，之后程序继续并记录演示完成。

JUL 常见级别从更详细到更严重包括 `FINEST`、`FINER`、`FINE`、`CONFIG`、`INFO`、`WARNING`、`SEVERE`。实际记录还受 logger 和 handler 的级别与配置影响；把 logger 调成 FINE 并不保证 handler 一定输出 FINE。

## 工作中怎么用

记录可用于关联请求或定位故障的安全标识、失败原因和异常堆栈；避免记录密码、令牌或未经处理的敏感数据。不要在所有调用层重复记录同一异常，也不要仅记录异常消息就丢失堆栈。日志级别和输出位置由应用配置管理；高频日志要考虑数据量和性能。

异常被捕获后是否继续应由业务决定。本例只是教学演示，不能推断真实订单处理已经成功。JUL 与 SLF4J 占位符、后端和配置不同，混用时先检查桥接与依赖。

## 手练与答案

1. 将 `Level.INFO` 改成 `Level.FINE`：在默认常见配置下该消息可能不显示，具体取决于 logger 和 handler 的级别。
2. 将 `logger.log(Level.WARNING, "订单处理失败", ex)` 改成只记录消息：异常堆栈不再随这条日志提供，排查信息减少。

参考：[廖雪峰：JDK Logging](https://liaoxuefeng.com/books/java/exception/jdk-logging/index.html)；[Oracle Logger API](https://docs.oracle.com/en/java/javase/17/docs/api/java.logging/java/util/logging/Logger.html)。
