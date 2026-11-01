# Reader

> **重要程度：高**｜**手练：需要**
> **重点**：字符读取、编码、流末尾与缓冲。

## 解决什么问题

`Reader` 读取字符序列；处理文本时，比直接把每个字节当字符更合适。文件和网络的字节仍需按指定编码解码为字符。`StringReader` 从已有字符串读取，不涉及文件编码。

## 完整例子

保存为 `ReaderDemo.java`，编译运行；日志写到标准错误。

```java
import java.io.Reader;
import java.io.StringReader;
import java.util.logging.Logger;
public class ReaderDemo {
    private static final Logger logger = Logger.getLogger(ReaderDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：读取字符数据
         * ========================================================================
         * 数据源：内存中的示例文本
         * 操作：
         * 1) 读取两个字符单元
         * 2) 检查流结束标记
         */
        logger.info("开始读取文本");
        // 1.1 创建字符输入流
        try (Reader reader = new StringReader("A中")) {
            // 1.2 读取并显示字符及结束标记
            System.out.println("first=" + (char) reader.read());
            System.out.println("second=" + (char) reader.read());
            System.out.println("end=" + reader.read());
        }
        logger.info("文本读取完成");
    }
}
```

标准输出：

```text
first=A
second=中
end=-1
```

## 执行过程

StringReader 依次读取 `A`、`中`，第三次读取到末尾返回 -1。单次 `read()` 返回 int：有效值对应一个 UTF-16 代码单元，-1 表示结束。示例在已知有两个基本字符时直接强转 char；真实循环应先判断 -1 再转换，避免把结束标记当字符。

## 文件与编码

从字节流读 UTF-8 文本时可用 `InputStreamReader` 指定 `StandardCharsets.UTF_8`；读取文件也可用 `Files.newBufferedReader(path, StandardCharsets.UTF_8)`。缓冲读取能减少底层频繁操作；性能取决于场景。`BufferedReader.readLine()` 返回一行且不包含行终止符，到末尾返回 null，空行则是空字符串，两者不同。

Java 字符读取按 UTF-16 单元工作；补充平面的码点可能由两个 char 表示。若业务要求按 Unicode 码点或用户感知字符处理，需选择对应 API 和规则。关闭自己打开的 reader；处理无效编码、文件不存在等异常。

## 手练与答案

把输入改为空字符串：第一次 `read()` 返回 -1。用 `readLine()` 读取一行空行得到 `""`；读到末尾得到 null，不能把两者混为一谈。

参考：[廖雪峰：Reader](https://liaoxuefeng.com/books/java/io/reader/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Reader.html)。
