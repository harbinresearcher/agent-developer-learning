# File对象

> **重要程度：高**｜**手练：需要**
> **重点**：路径与文件对象、相对路径、存在性。

## 先理解：File 是路径表示

`java.io.File` 表示文件或目录的抽象路径；`new File(...)` 只创建 Java 对象，不自动在磁盘创建文件。它能查询部分属性、列出目录或请求创建和删除；现代代码也常用 `Path` 与 `Files`，参见 [[使用Files]]。

## 完整例子

保存为 `FileInfoDemo.java`，编译运行；日志写到标准错误。例子不读取或创建磁盘文件。

```java
import java.io.File;
import java.util.logging.Logger;
public class FileInfoDemo {
    private static final Logger logger = Logger.getLogger(FileInfoDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：检查路径信息
         * ========================================================================
         * 数据源：一个相对路径
         * 操作：
         * 1) 创建 File 路径对象
         * 2) 读取名称及路径属性
         */
        logger.info("开始检查路径");
        // 1.1 创建相对路径对象，不创建磁盘文件
        File file = new File("data", "report.txt");
        // 1.2 查询路径名称和是否为绝对路径
        System.out.println("name=" + file.getName());
        System.out.println("absolute=" + file.isAbsolute());
        System.out.println("absoluteForm=" + file.getAbsoluteFile().isAbsolute());
        logger.info("路径检查完成");
    }
}
```

标准输出：

```text
name=report.txt
absolute=false
absoluteForm=true
```

## 执行过程

`new File("data", "report.txt")` 表示相对于进程工作目录的路径。`getName()` 返回最后一段 `report.txt`；`isAbsolute()` 为 false。`getAbsoluteFile()` 生成绝对形式的路径对象，所以第三行是 true，但它不证明目标存在。

相对路径基于进程工作目录，而不保证是源码、class 或项目根目录。路径分隔符因系统而异，使用 `File(parent, child)` 或 `Path.resolve` 组合，避免手工拼接平台分隔符。

## 存在、类型与错误

`exists()`、`isFile()`、`isDirectory()` 可查询当时状态；结果会随时间变化，不能用“先检查再操作”消除并发下的所有竞态。`listFiles()` 可能因不是目录或 I/O 错误返回 null，应区分并处理。部分 File 修改方法以 boolean 返回是否成功；不要忽略失败。

绝对路径、规范化路径和真实路径不是同义词；符号链接和文件系统权限会影响实际访问。不要把未经验证的外部路径直接用于删除、覆盖或解压，按业务限定目标目录并处理符号链接。

## 手练与答案

把文件名改成 `"notes.txt"`，name 输出 `notes.txt`；对象创建不会在磁盘生成该文件。`isAbsolute()` 返回 false 不代表文件不存在，只说明此路径形式不是绝对路径。

参考：[廖雪峰：File对象](https://liaoxuefeng.com/books/java/io/file/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/File.html)。
