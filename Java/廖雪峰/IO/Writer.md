# Writer

> **重要程度：高**｜**手练：需要**
> **重点**：字符写入、编码、关闭与 flush。

## 解决什么问题

`Writer` 写出字符数据；输出到文件或网络时，底层需要将字符按指定编码转换成字节。`StringWriter` 写入内存中的字符缓冲，适合演示和小型文本构造。

## 完整例子

保存为 `WriterDemo.java`，编译运行；日志写到标准错误。

```java
import java.io.StringWriter;
import java.io.Writer;
import java.util.logging.Logger;
public class WriterDemo {
    private static final Logger logger = Logger.getLogger(WriterDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：写入文本
         * ========================================================================
         * 数据源：两段示例字符串
         * 操作：
         * 1) 将文本写入字符流
         * 2) 检查内存中的结果
         */
        logger.info("开始写入文本");
        // 1.1 创建内存字符输出流
        StringWriter buffer = new StringWriter();
        try (Writer writer = buffer) {
            // 1.2 逐段写入并刷新
            writer.write("你好");
            writer.write(" Java");
            writer.flush();
            System.out.println("text=" + buffer.toString());
        }
        logger.info("文本写入完成");
    }
}
```

标准输出：

```text
text=你好 Java
```

## 执行过程

两次 `write` 依次写入“你好”和“ Java”。`flush` 请求把缓冲内容向目标推进，`buffer.toString()` 得到完整文本。try 结束后调用 close；StringWriter 的关闭行为较特殊，不能据此推断文件 Writer 关闭后仍可写。

## 文件与编码

写 UTF-8 文件可使用 `Files.newBufferedWriter(path, StandardCharsets.UTF_8)` 并按需求指定打开选项。打开选项决定创建、覆盖或追加，覆盖前应确认是否允许替换原文件；不要把教程示例中的路径当作安全的生产默认值。

`OutputStreamWriter` 将字符编码为字节，编码与读取端必须约定一致。`flush` 将缓冲向底层推进，不等于数据已持久化到物理设备；`close` 通常会刷新并关闭其管理的底层资源，具体以实现契约为准。错误可能在写入、刷新或关闭时发生。

## 工作中怎么用

根据所有权关闭流；大规模数据分块写入，限制资源占用。并发写文件需设计同步和输出顺序。写入敏感文本时控制权限与日志，不要误把秘密输出到共享文件。

## 手练与答案

把第二次写入改为 `"世界"`：输出 `text=你好世界`。如果文件读取端使用与写入端不同的编码，文本可能不能正确还原；StringWriter 本身不进行字节编码。

参考：[廖雪峰：Writer](https://liaoxuefeng.com/books/java/io/writer/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Writer.html)。
