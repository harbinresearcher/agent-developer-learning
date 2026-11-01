# 操作Zip

> **重要程度：高**｜**手练：需要**
> **重点**：ZIP 条目、流关闭、解压边界和资源限制。

## 解决什么问题

ZIP 归档包含一个或多个条目，每个条目有名称和内容。Java 的 `ZipOutputStream` 可以写条目，`ZipInputStream` 可以依次读取；条目名和内容都是输入数据，不能直接信任。

## 完整例子

保存为 `ZipMemoryDemo.java`，用 Java 11 或更高版本编译运行；日志写到标准错误。例子只在内存生成和读取受控内容，不向文件系统解压。

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.nio.charset.StandardCharsets;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;
import java.util.zip.ZipOutputStream;
import java.util.logging.Logger;
public class ZipMemoryDemo {
    private static final Logger logger = Logger.getLogger(ZipMemoryDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：创建并读取内存 ZIP
         * ========================================================================
         * 数据源：一个受控的文件名和短文本
         * 操作：
         * 1) 写入一个 ZIP 条目
         * 2) 读取条目名和文本
         */
        logger.info("开始处理 ZIP");
        // 1.1 创建仅存在于内存中的压缩包
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        try (ZipOutputStream zip = new ZipOutputStream(buffer)) {
            zip.putNextEntry(new ZipEntry("hello.txt"));
            zip.write("你好".getBytes(StandardCharsets.UTF_8));
            zip.closeEntry();
        }
        // 1.2 读取本例生成的条目
        try (ZipInputStream zip = new ZipInputStream(new ByteArrayInputStream(buffer.toByteArray()))) {
            ZipEntry entry = zip.getNextEntry();
            if (entry == null) throw new IllegalStateException("缺少条目");
            System.out.println("name=" + entry.getName());
            System.out.println("text=" + new String(zip.readAllBytes(), StandardCharsets.UTF_8));
        }
        logger.info("ZIP 处理完成");
    }
}
```

标准输出：

```text
name=hello.txt
text=你好
```

## 执行过程

`putNextEntry` 创建 `hello.txt` 条目，`write` 把 UTF-8 文本写入，`closeEntry` 结束条目；关闭 ZipOutputStream 完成归档。随后 ZipInputStream 的 `getNextEntry` 读取第一条，检查非 null 后读取该条目的数据并解码。

示例使用 `readAllBytes` 是因为条目内容完全受控且很小。真实归档应逐项循环并分块读取，限制条目数量、单条与总解压大小、路径长度和处理时间；压缩包很小不代表解压后也小。

## 解压到磁盘的安全边界

不能直接把条目名称拼接到输出目录后写文件：`../`、绝对路径或其他路径形式可能逃出目标目录。应拒绝不安全名称，规范化并确认目标仍在允许目录，同时处理符号链接与竞态；仅用字符串前缀比较并不足以完整保证安全。不要无条件覆盖已有用户文件。

## 工作中怎么用

指定字符编码和覆盖策略，及时关闭归档与条目流；处理格式错误、重复条目和 I/O 失败。安全要求高时优先采用维护良好的解压组件并审查其路径和资源限制行为，不把教学示例直接用于处理不可信文件。

## 手练与答案

条目名改为 `"note.txt"`：第一行输出 `name=note.txt`。删除写入内容而保留条目：文本输出为空；若没有条目，`getNextEntry()` 返回 null，示例抛出明确错误。

参考：[廖雪峰：操作Zip](https://liaoxuefeng.com/books/java/io/zip/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/zip/ZipInputStream.html)。
