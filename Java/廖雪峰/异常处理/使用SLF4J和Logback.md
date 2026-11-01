# 使用 SLF4J 和 Logback

> **重要程度：高**｜**手练：需要**
> **重点**：日志门面与实现、占位符、异常堆栈、provider 和版本兼容。

## 先理解：谁定义接口，谁真正输出

SLF4J 提供统一的日志 API（门面）；Logback 是可以接收并输出日志的实现。业务代码调用 SLF4J API，运行时由匹配的 provider 连接到 Logback。没有匹配 provider 时，SLF4J 可能警告并使用不输出日志的空实现，光有 `slf4j-api` 不等于日志已经写出。

## 完整例子

以下 Java 代码依赖 SLF4J API。准备兼容的 SLF4J 和 Logback 依赖后，保存为 `Slf4jDemo.java` 并编译运行：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Slf4jDemo {
    private static final Logger logger = LoggerFactory.getLogger(Slf4jDemo.class);
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：记录订单处理信息
         * ========================================================================
         * 数据源：本地订单编号和模拟异常
         * 操作：
         * 1) 用占位符记录业务标识
         * 2) 记录模拟异常及堆栈
         */
        logger.info("开始处理订单");
        // 1.1 记录安全的订单编号
        int orderId = 42;
        logger.info("订单编号: {}", orderId);
        // 1.2 保留异常对象及堆栈
        try {
            throw new IllegalStateException("模拟失败");
        } catch (IllegalStateException ex) {
            logger.error("订单处理失败，订单编号: {}", orderId, ex);
        }
        logger.info("订单处理演示完成");
    }
}
```

下面是 Maven `pom.xml` 中的依赖片段，放在 `<dependencies>` 内；它不是完整项目文件。例子使用 SLF4J 2.0.20 与 Logback 1.5.38，项目内已有依赖管理时以项目兼容版本为准。该示例已用本机缓存的对应 jar 在 Java 11 下编译运行；此依赖片段未单独执行 Maven 构建。

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.20</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.38</version>
    <scope>runtime</scope>
</dependency>
```

日志应包含订单编号 42、模拟失败及 `IllegalStateException` 堆栈，以及演示结束信息。具体时间、输出流和格式取决于 Logback 配置。项目可通过 Maven 正常编译并配置应用运行类路径；直接运行 `java Slf4jDemo` 时，还必须让运行时类路径包含 API、Logback Classic 和 Logback Core 的 jar。

## 执行过程

`LoggerFactory.getLogger` 创建与类关联的门面日志器。`logger.info("订单编号: {}", orderId)` 用 SLF4J 的 `{}` 占位符填入参数。模拟操作失败后，`logger.error("订单处理失败，订单编号: {}", orderId, ex)` 将最后的异常对象作为异常参数，输出日志和堆栈，而不是只输出异常消息。捕获后继续只是教学演示，真实业务必须正确处理失败。

SLF4J 的参数占位符与 JDK Logging 的 `{0}` 不同。参数化日志避免在不需要输出时先做部分字符串拼接，但传给方法的表达式本身仍会求值；耗时计算要按需要用日志级别判断或 API 支持的惰性方式。

## 版本与排错

SLF4J 2.x 用 `ServiceLoader` 查找 provider；为 1.7 及更早版本准备的旧 binding 不能当作 2.x provider 使用。遇到“找不到 provider”、日志不输出或重复输出时，先检查依赖树、API 与 provider 版本、是否同时存在多个实现或循环桥接，再检查日志级别与配置。不要同时引入多个无关日志后端。

Logback 配置控制 logger 级别、输出目的地和格式。应用负责选择和配置实际后端；供别人使用的库通常只依赖日志门面，不应强制替应用挑选 provider。

## 工作中怎么用

日志里记录定位所需的安全标识、操作结果和异常堆栈；不要记录密码、令牌或完整敏感数据。跨层传递异常时避免每层重复记录。高频日志需结合负载、存储与隐私要求控制级别和采样。

## 手练与答案

1. 将级别调到 ERROR：INFO 日志可能被过滤，ERROR 仍需同时满足 logger 与输出端配置。
2. 从运行类路径移除 Logback provider：SLF4J 2.x 通常警告找不到 provider 并回退到空实现，应用业务仍可能运行，但日志不会正常输出。
3. 将异常参数去掉：日志失去原异常堆栈，定位错误的信息减少。

参考：[廖雪峰：SLF4J和Logback](https://liaoxuefeng.com/books/java/exception/slf4j-logback/index.html)；[SLF4J 错误说明](https://www.slf4j.org/codes.html)；[SLF4J 手册](https://www.slf4j.org/manual.html)。
