# OutputStream

> **重要程度：高**｜**手练：需要**
> **重点**：写入字节、flush 与 close、数据完整性。

## 解决什么问题

`OutputStream` 是字节输出流的抽象，可向文件、网络或内存写字节。先弄清数据格式，再按正确方式编码或写入；输出流不会自动把任意对象转成业务需要的格式。

## 完整例子

保存为 `OutputStreamDemo.java`，编译运行；日志写到标准错误。示例写入内存，不创建磁盘文件。

```java
import java.io.ByteArrayOutputStream;
import java.io.OutputStream;
import java.util.Arrays;
import java.util.logging.Logger;
public class OutputStreamDemo {
    private static final Logger logger = Logger.getLogger(OutputStreamDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：写入示例字节
         * ========================================================================
         * 数据源：三个整数值对应的字节
         * 操作：
         * 1) 创建输出流并写入字节
         * 2) 刷新并检查结果
         */
        logger.info("开始写入字节");
        // 1.1 创建内存输出流
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        try (OutputStream out = buffer) {
            // 1.2 写入完整字节数组并刷新
            out.write(new byte[]{10, 20, 30});
            out.flush();
            System.out.println("bytes=" + Arrays.toString(buffer.toByteArray()));
        }
        logger.info("字节写入完成");
    }
}
```

标准输出：

```text
bytes=[10, 20, 30]
```

## 执行过程

`ByteArrayOutputStream` 接收写入的字节，变量 `out` 以 `OutputStream` 接口操作它。`write` 把三个字节写入，`flush` 请求将流中缓冲数据向目标推进，`toByteArray()` 返回当前字节副本。退出 try 时调用 close。

内存流不代表文件或网络流的所有行为：真实资源可能因权限、磁盘空间、网络等问题抛出 `IOException`。关闭流和错误传播要按具体资源与所有权处理。

## flush 与持久化

`flush()` 不是 `close()`：它请求写出当前缓冲，但流通常仍可继续使用。`close()` 结束流并释放相关资源。对文件流，flush 把数据推向底层不等于保证已经持久写入物理设备；需要崩溃一致性时，按存储 API 和系统语义设计同步与原子替换。

用 `try-with-resources` 关闭由当前方法创建且负责拥有的流；不要擅自关闭调用方传入并仍要使用的流。文本写出时应指定编码，或使用适当的 Writer。

## 手练与答案

改为 `out.write(new byte[]{10, 20, 30}, 1, 2)`：输出 `[20, 30]`。删除本例 flush 后，内存流仍可能得到相同输出；不能据此推断缓冲文件或网络流也不需要按契约刷新。

参考：[廖雪峰：OutputStream](https://liaoxuefeng.com/books/java/io/output-stream/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/OutputStream.html)。
