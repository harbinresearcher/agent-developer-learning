# 使用 Commons Logging

> **重要程度：中**｜**手练：按需**
> **重点**：日志门面、后端发现、异常堆栈、桥接冲突。

## 先理解：门面与实现

Apache Commons Logging（JCL）提供 `Log` 和 `LogFactory`，让代码通过统一接口发日志。实际日志输出交给发现到的实现；使用哪个后端、级别和格式由依赖与配置决定。阅读旧项目时先查清这个链路，再修改配置。

## 完整例子

保存为 `CommonsLogDemo.java`。例子需要 Commons Logging 依赖，不能仅凭 JDK 编译；日志格式和输出位置取决于实际后端。

```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
public class CommonsLogDemo {
    private static final Log logger = LogFactory.getLog(CommonsLogDemo.class);
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：记录任务结果
         * ========================================================================
         * 数据源：演示任务和模拟异常
         * 操作：
         * 1) 记录任务开始
         * 2) 记录失败原因及原始异常
         */
        logger.info("开始处理任务");
        // 1.1 模拟任务失败并记录异常
        try {
            throw new IllegalStateException("模拟失败");
        } catch (IllegalStateException ex) {
            logger.error("任务处理失败", ex);
        }
        // 1.2 结束演示
        logger.info("任务演示完成");
    }
}
```

Maven 项目的 `<dependencies>` 中可以声明以下依赖；这是依赖片段，不是完整 `pom.xml`：

```xml
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.3.6</version>
</dependency>
```

示例已使用本机 `commons-logging-1.3.6.jar` 与 Java 11 编译运行；此 Maven 片段未单独构建。日志包含开始、失败、`IllegalStateException: 模拟失败` 的堆栈和演示结束。若直接用 `java` 命令运行，需在类路径中包含编译产物和该 jar。

## 执行过程

`LogFactory.getLog` 为类取得 `Log`；`info` 记录任务状态。模拟异常被捕获后，`logger.error("任务处理失败", ex)` 将异常对象交给门面，后端可记录堆栈。程序继续只是为了展示调用，真实失败是否继续由业务决定。

Commons Logging 会根据其版本、类路径和配置选择日志实现；只看源码里的 `Log` 不足以判断最终输出到了哪里。Spring 也有自己的 `spring-jcl` 桥接机制，不应把它与独立 Commons Logging 工件的发现行为一概而论；按实际项目版本和依赖树检查。

## 工作中怎么用

先找到哪一个库引入 JCL、实际使用什么实现，再检查级别和配置。将旧 API 桥接到统一日志门面时，避免同时配置相反方向的桥接，造成循环；不要随意加入多个冲突实现。需要保留异常堆栈，但不应在日志中输出密码、令牌或完整敏感数据。

## 手练与答案

1. 去掉 `logger.error` 的异常参数：仍能记录错误消息，但原始异常堆栈不会通过这次调用传给后端。
2. 在项目中添加其他日志实现后输出变化：应先检查类路径和后端选择，不应断定 JCL 自身把日志固定写到某个文件。

参考：[廖雪峰：Commons Logging](https://liaoxuefeng.com/books/java/exception/commons-logging/index.html)；[Apache Commons Logging 用户指南](https://commons.apache.org/proper/commons-logging/guide.html)。
