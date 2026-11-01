# 使用Files

> **重要程度：高**｜**手练：需要**
> **重点**：Path、UTF-8 读写、打开选项和资源清理。

## 解决什么问题

`Path` 表示路径，`Files` 提供常用文件操作。Java 11 起可用 `readString`、`writeString` 简化小型文本文件的读写；读取与写入都应明确编码和文件覆盖策略。

## 完整例子

保存为 `FilesDemo.java`，用 Java 11 或更高版本编译运行；日志写到标准错误。示例只创建并删除自己的临时文件。

```java
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.logging.Logger;
public class FilesDemo {
    private static final Logger logger = Logger.getLogger(FilesDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：读写临时文本文件
         * ========================================================================
         * 数据源：程序创建的临时文件与示例文本
         * 操作：
         * 1) 以 UTF-8 写入并读取
         * 2) 删除本次创建的临时文件
         */
        logger.info("开始处理临时文件");
        // 1.1 创建本次示例专属文件
        Path file = Files.createTempFile("java-notes-", ".txt");
        try {
            // 1.2 明确 UTF-8 写入和读取
            Files.writeString(file, "你好 Java", StandardCharsets.UTF_8);
            String text = Files.readString(file, StandardCharsets.UTF_8);
            System.out.println("text=" + text);
        } finally {
            // 1.3 清理本次创建的文件
            Files.deleteIfExists(file);
        }
        logger.info("临时文件处理完成");
    }
}
```

标准输出：

```text
text=你好 Java
```

## 执行过程

`createTempFile` 创建一个新临时文件并返回 Path。`writeString` 以 UTF-8 写入示例文本，`readString` 使用相同编码读取，输出还原的内容。finally 中只删除本次创建的路径，即使读写抛异常也尝试清理；如果删除本身失败，调用方需要处理该异常。

## 边界和工作用法

`readString` 会读取整个文件到内存，适合有明确大小上限的小文件。大文件可使用带缓冲的 Reader 或流式处理，并在退出时关闭资源。`Files.lines` 返回的 Stream 也需要关闭。

`writeString` 在未提供选项时按 API 默认策略创建或截断文件；对用户已有文件写入前必须确认覆盖、追加或创建失败等策略。路径来自外部时，先限定允许目录并考虑符号链接；规范化路径不是完整的授权检查。

使用临时文件再移动可帮助设计替换流程，但能否原子移动取决于文件系统和选项。真正要求崩溃持久性时，还需考虑同步、异常恢复和平台语义，不能把一次成功的 write 当成完整事务。

## 手练与答案

把写入文本改成 `"Ada"`：读回输出 `text=Ada`。如果删除 `finally` 而读取发生异常：本例创建的临时文件可能残留。

参考：[廖雪峰：使用Files](https://liaoxuefeng.com/books/java/io/files/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/file/Files.html)。
