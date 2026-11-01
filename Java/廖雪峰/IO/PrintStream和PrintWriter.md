# PrintStream和PrintWriter

> **重要程度：高**｜**手练：需要**
> **重点**：文本输出、自动换行、错误检查与资源所有权。

## 解决什么问题

`PrintStream` 与 `PrintWriter` 提供方便的 `print`、`println`、`printf`。`System.out` 是 PrintStream。它们适合按格式输出文本，但错误处理方式与普通 Writer 不完全一样。

## 完整例子

保存为 `PrintDemo.java`，编译运行；日志写到标准错误。示例不关闭共享的 `System.out`。

```java
import java.io.IOException;
import java.io.PrintWriter;
import java.io.StringWriter;
import java.io.Writer;
import java.util.logging.Logger;
public class PrintDemo {
    private static final Logger logger = Logger.getLogger(PrintDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：输出文本并检查错误
         * ========================================================================
         * 数据源：内存字符输出与模拟失败的写入器
         * 操作：
         * 1) 输出一条文本
         * 2) 检查写入失败状态
         */
        logger.info("开始输出文本");
        // 1.1 向内存输出文本
        StringWriter buffer = new StringWriter();
        PrintWriter writer = new PrintWriter(buffer);
        writer.print("你好");
        writer.flush();
        System.out.println("text=" + buffer.toString());
        // 1.2 模拟写入失败并检查错误标记
        PrintWriter broken = new PrintWriter(new FailingWriter());
        broken.print("数据");
        System.out.println("error=" + broken.checkError());
        broken.close();
        writer.close();
        logger.info("文本输出完成");
    }
    static class FailingWriter extends Writer {
        public void write(char[] data, int offset, int length) throws IOException {
            throw new IOException("模拟写入失败");
        }
        public void flush() { }
        public void close() { }
    }
}
```

标准输出：

```text
text=你好
error=true
```

## 执行过程

第一个 PrintWriter 向 StringWriter 写入“你好”，刷新后输出内存结果。第二个 PrintWriter 包装一个故意在 write 时抛 `IOException` 的 Writer；`print` 没有把该 IOException 直接抛给调用方，而是设置内部错误状态，`checkError()` 返回 true。`checkError()` 也会刷新流。

## 输出和资源边界

`print` 不自动换行；`println` 输出换行；`printf` 按格式输出，`%n` 使用平台换行符。自动刷新只在构造器启用且符合该类文档列出的特定操作时发生，不能认为每次 print 都自动刷新。

PrintWriter 和 PrintStream 的常见写方法不会像普通 Writer 那样直接向调用方抛出 IO 异常，重要输出需要检查错误或使用能够显式传播异常的 API。输出到字节或文件时，明确指定字符编码，并遵循目标 API 的构造器规则。

关闭包装对象可能关闭底层流；`System.out` 是进程共享资源，普通业务代码不应随手关闭它。用户文件的覆盖和追加选项需明确，敏感数据不要直接输出到日志。

## 手练与答案

把模拟 Writer 改为正常写入：`checkError()` 可为 false。把 `print("你好")` 改成 `println("你好")`：缓冲区内容末尾增加平台换行符。

参考：[廖雪峰：PrintStream和PrintWriter](https://liaoxuefeng.com/books/java/io/print/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/PrintWriter.html)。
