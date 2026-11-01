# InputStream

> **重要程度：高**｜**手练：需要**
> **重点**：字节读取、实际读取数量、结束标记和资源关闭。

## 解决什么问题

`InputStream` 是字节输入流的抽象：可以从文件、网络、内存等来源读取字节。它本身不理解这些字节是文本、图片还是压缩数据。读取完成后要根据资源所有权关闭真实流。

## 完整例子

保存为 `InputStreamDemo.java`，编译运行；日志写到标准错误。例子用内存流，避免依赖外部文件。

```java
import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.util.Arrays;
import java.util.logging.Logger;
public class InputStreamDemo {
    private static final Logger logger = Logger.getLogger(InputStreamDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：分块读取字节
         * ========================================================================
         * 数据源：三个示例字节
         * 操作：
         * 1) 使用固定缓冲区读取
         * 2) 只处理每次实际读取的字节
         */
        logger.info("开始读取字节");
        // 1.1 创建内存输入流与缓冲区
        byte[] data = {10, 20, 30};
        try (InputStream in = new ByteArrayInputStream(data)) {
            byte[] buffer = new byte[2];
            int count;
            // 1.2 循环读取，直到流结束
            while ((count = in.read(buffer)) != -1) {
                System.out.println("read=" + Arrays.toString(Arrays.copyOf(buffer, count)));
            }
        }
        logger.info("字节读取完成");
    }
}
```

标准输出：

```text
read=[10, 20]
read=[30]
```

## 执行过程

数据是 10、20、30 三个字节，缓冲区容量为 2。第一次读取两个字节，count 为 2；第二次读取一个字节，count 为 1；再读返回 -1，循环结束。第二次只处理前一个有效字节，不能误把缓冲区剩余的旧数据也当作新数据。

`read(byte[])` 返回实际读取数量，不保证每次填满缓冲区；非零长度缓冲区读到流末尾时返回 -1。单字节 `read()` 返回 0～255 的 int，-1 代表结束，不能先强转成 byte 再判断。真实流可能阻塞或抛 `IOException`。

## 工作中怎么用

用 `try-with-resources` 关闭方法负责拥有的文件或网络流；不要随意关闭由调用方继续使用的共享流。`available()` 不是文件总长度，也不能用它判断完整数据是否已接收。大文件应分块处理并限制输入大小；`readAllBytes()` 简单但会把剩余数据放入内存。

文本需要用明确字符集的 `InputStreamReader` 解码；UTF-8 字符可能跨缓冲块，不能把每块字节随意分别转成字符串再拼接而忽略边界。二进制数据不要按文本解码。

## 手练与答案

把缓冲区长度改为 4：这个内存流示例首次读取三个字节，输出 `[10, 20, 30]`。实际其他流仍不保证一次填满。把 `count` 忽略并总打印整个 buffer：最后一块可能混入旧字节。

参考：[廖雪峰：InputStream](https://liaoxuefeng.com/books/java/io/input-stream/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/InputStream.html)。
