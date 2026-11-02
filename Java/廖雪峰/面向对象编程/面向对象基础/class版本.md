# class 版本

> **重要程度：高**｜**手练：需要**
> **重点**：编译目标、class 主版本、运行时版本和依赖兼容。

## 为什么会出现版本错误

`javac` 将 Java 源码编译为 `.class` 字节码；class 文件包含版本信息。运行时 JVM 必须支持该版本，否则可能在加载时抛出 `UnsupportedClassVersionError`。例如 Java 11 对应主版本 55，Java 17 对应主版本 61。

编译器版本和运行时版本可能来自不同安装目录。先查看 `javac -version` 与 `java -version`，还要检查 Maven、IDE 和部署环境实际使用的 JDK。

## 完整例子

保存为 `VersionDemo.java`：

```java
import java.util.logging.Logger;
public class VersionDemo {
    private static final Logger logger = Logger.getLogger(VersionDemo.class.getName());
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：验证编译目标
         * 数据源：当前运行的 class 文件
         * 操作：1) 启动程序 2) 输出验证消息
         * ======================================================================== */
        logger.info("开始版本示例");
        // 1.1 输出用于比较的固定内容
        System.out.println("版本兼容示例");
        logger.info("版本示例完成");
    }
}
```

用 JDK 17 的 `javac` 和 `javap`，在文件所在目录运行：

```powershell
javac --release 11 -encoding UTF-8 -d out11 VersionDemo.java
javap -verbose out11/VersionDemo.class | Select-String "major version"
java -cp out11 VersionDemo
javac --release 17 -encoding UTF-8 -d out17 VersionDemo.java
javap -verbose out17/VersionDemo.class | Select-String "major version"
```

两次 `javap` 分别显示主版本 `55` 和 `61`；兼容运行时执行第一份 class 会输出 `版本兼容示例`。若实际用 Java 11 运行 `out17` 中的类，会因不支持主版本 61 而失败；用 Java 17 运行则不能复现这一错误。

## `--release` 做什么

在支持目标版本的 JDK 上，`--release 11` 限制语言规则、生成目标和可用的对应标准 API。它不能把 Java 17 特性自动转换成 Java 11 能理解的代码，也不能保证第三方依赖兼容 Java 11。

项目排错时同时检查应用、每个依赖的 class 版本、目标 JDK 的 API 与实际运行时。预览特性还需要匹配对应版本和启用方式；仅看主版本不足以证明整个项目可运行。

## 动手与自查

1. 分别编译为 11 和 17，检查两份 class 的主版本。
2. 在隔离环境用 Java 11 运行 17 目标文件，记录异常；再用 Java 17 运行。
3. 为什么应用自身目标版本是 11，仍可能因某个依赖不兼容而启动失败？

来源：[教程：class版本](https://liaoxuefeng.com/books/java/oop/basic/class-version/index.html)、[javac 选项](https://docs.oracle.com/en/java/javase/17/docs/specs/man/javac.html)。
