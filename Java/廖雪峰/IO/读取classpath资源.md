# 读取classpath资源

> **重要程度：高**｜**手练：需要**
> **重点**：类路径资源、流读取、JAR 内资源与缺失处理。

## 解决什么问题

应用可能需要读取随程序一起打包的配置模板或静态数据。类路径资源通过类加载器查找；打包进 JAR 后它不一定是可直接访问的普通文件，所以优先按资源流读取。

## 完整例子

将代码保存为 `ResourceDemo.java`，在同一目录创建 UTF-8 文本 `message.txt`，内容为 `你好`。在该目录编译 `javac -encoding UTF-8 ResourceDemo.java`，再运行 `java -cp . ResourceDemo`；日志写到标准错误。例子假定 `.` 是包含 class 与资源的类路径根目录。

```java
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.util.logging.Logger;
public class ResourceDemo {
    private static final Logger logger = Logger.getLogger(ResourceDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：读取类路径资源
         * ========================================================================
         * 数据源：类路径根目录中的 message.txt
         * 操作：
         * 1) 查找资源并检查是否存在
         * 2) 用 UTF-8 读取文本
         */
        logger.info("开始读取资源");
        // 1.1 从类路径根目录获取资源流
        try (InputStream in = ResourceDemo.class.getResourceAsStream("/message.txt")) {
            if (in == null) throw new IllegalStateException("缺少 message.txt 资源");
            // 1.2 按 UTF-8 解码受控的少量内容
            String text = new String(in.readAllBytes(), StandardCharsets.UTF_8).strip();
            System.out.println("text=" + text);
        }
        logger.info("资源读取完成");
    }
}
```

标准输出：

```text
text=你好
```

## 执行过程

`ResourceDemo.class.getResourceAsStream("/message.txt")` 中的前导斜杠表示从类路径根目录查找。找到资源时返回输入流；找不到返回 null，示例明确抛出错误而不继续解引用。`readAllBytes` 适用于此受控的小资源，随后按 UTF-8 解码，`strip()` 去掉两端空白，输出“你好”。

## 路径和打包边界

`Class.getResourceAsStream` 的无前导斜杠名称相对该类所在包；带斜杠从类路径根查找。`ClassLoader.getResourceAsStream` 的名称约定不同，通常不加前导斜杠，应查所用 API。

资源实际可能位于 JAR 内，资源 URL 不保证能转换成普通文件路径。不要把源码目录或 IDE 工作目录当成部署后的资源路径。构建工具要把资源放进产物，测试也应覆盖打包后的运行方式；例子仅验证类路径目录方式。

资源流应关闭；不要无条件把大资源一次读入内存。资源不存在、编码不一致或打包漏掉资源都应给出可定位的错误。包含密码的配置不适合直接作为公开打包资源。

## 手练与答案

移走 `message.txt` 再运行：示例抛出“缺少 message.txt 资源”。把资源放进 `config` 子目录，并将路径改为 `/config/message.txt`：类路径中相应位置存在文件时可读取。

参考：[廖雪峰：读取classpath资源](https://liaoxuefeng.com/books/java/io/classpath/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Class.html)。
