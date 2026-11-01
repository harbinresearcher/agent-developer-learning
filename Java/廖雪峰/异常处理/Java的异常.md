# Java的异常

> **重要程度：高**｜**手练：需要**
> **重点**：异常传播、受检和非受检异常、定位堆栈。

## 为什么需要异常

程序执行时可能遇到无效输入、文件不存在等情况。异常让当前正常流程中断，并把错误信息沿调用链传给能够处理它的代码。异常不等同于程序必然退出：调用方捕获后可以按业务决定如何处理。

## 完整例子

保存为 `ExceptionBasicsDemo.java`，使用 `javac -encoding UTF-8 ExceptionBasicsDemo.java` 编译，运行 `java ExceptionBasicsDemo`。日志写到标准错误。

```java
import java.util.logging.Logger;
public class ExceptionBasicsDemo {
    private static final Logger logger = Logger.getLogger(ExceptionBasicsDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：解析数据并处理格式错误
         * ========================================================================
         * 数据源：两条待解析的文本
         * 操作：
         * 1) 调用解析方法
         * 2) 在调用方捕获并输出错误类型
         */
        logger.info("开始解析数据");
        // 1.1 准备正常和异常输入
        String[] inputs = {"12", "abc"};
        // 1.2 逐条解析并处理异常
        for (String input : inputs) {
            try {
                System.out.println("value=" + parse(input));
            } catch (NumberFormatException ex) {
                System.out.println("error=" + ex.getClass().getSimpleName());
            }
        }
        logger.info("数据解析完成");
    }
    static int parse(String text) {
        return Integer.parseInt(text);
    }
}
```

标准输出：

```text
value=12
error=NumberFormatException
```

## 按调用顺序看

1. 输入 `"12"` 时，`parse` 调用 `Integer.parseInt` 并返回整数，调用方输出 12。
2. 输入 `"abc"` 时，`parseInt` 抛出 `NumberFormatException`；`parse` 没有捕获，方法提前结束，异常传回调用方。
3. 调用方的 `catch` 匹配该类型，输出异常类名；循环可以继续。本例选择逐条处理，实际业务不能因此假定任何错误都可忽略。

## 异常类型和编译规则

`Throwable` 是异常与错误的共同父类。`Exception` 下既有受检异常，也有 `RuntimeException` 及其子类。`IOException` 等受检异常通常要求调用方捕获或在方法签名中声明；`NumberFormatException`、`NullPointerException` 属于运行时异常，编译器不要求这样做。`Error` 通常表示严重的运行环境问题，不应作为普通业务校验手段。

异常对象可以携带类型、消息、调用堆栈、cause 和 suppressed 异常。堆栈帮助定位出错调用，但消息可能包含输入或敏感信息；记录前应遵循项目的数据保护约定。捕获后应明确恢复、转换并保留原因，或向上抛出；空 catch 会掩盖失败。

## 工作中怎么用

对可预期的非法输入优先做明确校验，并返回清晰的错误；对文件或网络等故障按业务处理。不要用异常作为普通循环分支，也不要把所有异常一律转换成“成功”。详见 [[捕获异常]]、[[抛出异常]]。

## 手练与答案

将 `"abc"` 改为 `"30"`：两次都输出 value。删掉调用方的 catch：第二次解析产生未处理异常，当前线程的正常执行中断，后续代码不会按原路径执行。

参考：[Oracle：异常概述](https://docs.oracle.com/javase/tutorial/essential/exceptions/index.html)。

课程原文：[廖雪峰：Java的异常](https://liaoxuefeng.com/books/java/exception/java-exception/index.html)。
