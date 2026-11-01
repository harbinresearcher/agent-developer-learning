# Filter模式

> **重要程度：高**｜**手练：需要**
> **重点**：流包装、缓冲、关闭链与职责。

## 解决什么问题

流包装把一个已有流交给另一个流，在保持类似接口的同时增加缓冲、计数、解码或其他职责。这样的组合常被称为装饰器或过滤流；功能与关闭行为由具体包装类决定。

## 完整例子

保存为 `FilterStreamDemo.java`，编译运行；日志写到标准错误。

```java
import java.io.BufferedInputStream;
import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.util.logging.Logger;
public class FilterStreamDemo {
    private static final Logger logger = Logger.getLogger(FilterStreamDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：包装并读取输入流
         * ========================================================================
         * 数据源：内存中的字节
         * 操作：
         * 1) 为原始输入流添加缓冲包装
         * 2) 读取数据并关闭外层流
         */
        logger.info("开始读取包装流");
        // 1.1 包装原始输入流
        InputStream raw = new ByteArrayInputStream(new byte[]{65, 66});
        try (InputStream buffered = new BufferedInputStream(raw)) {
            // 1.2 通过包装流读取
            System.out.println("first=" + buffered.read());
            System.out.println("second=" + buffered.read());
            System.out.println("end=" + buffered.read());
        }
        logger.info("包装流读取完成");
    }
}
```

标准输出：

```text
first=65
second=66
end=-1
```

## 执行过程

原始 `ByteArrayInputStream` 提供 65 和 66 两个字节。`BufferedInputStream` 包装它，调用外层 `read()` 仍依次得到 65、66，随后得到 -1。关闭 BufferedInputStream 会关闭它包装的输入流。本例是内存流，无法证明文件读取性能差异。

## 包装链与所有权

缓冲流可减少对底层资源的频繁调用，但不改变字节数据的业务含义。`InputStreamReader` 负责按字符集解码字节，`BufferedReader` 可再提供缓冲和按行读取。链上每一层的输入输出类型不同，应明确组合顺序与编码。

外层 close 是否关闭底层资源取决于具体实现；创建包装时要明确谁负责关闭。若底层资源由调用方共享，不能默认关闭包装层不会影响调用方。写出链中 flush 和 close 的传播、错误与顺序也要查相应 API。

## 工作中怎么用

按需要组合现成流，使用 try-with-resources；不要为了“使用设计模式”叠加无必要的层。涉及网络或大型文件时要限制大小、处理阻塞和异常，并用测试或测量判断缓冲策略。

## 手练与答案

把第二个字节改成 67：第二行输出 `second=67`。删除 BufferedInputStream 后，内存示例的读取结果仍相同；这不能证明所有真实流的性能或资源行为相同。

参考：[廖雪峰：Filter模式](https://liaoxuefeng.com/books/java/io/filter/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/BufferedInputStream.html)。
