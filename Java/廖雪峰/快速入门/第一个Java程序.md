# 第一个 Java 程序

> **重要程度：高**｜**手练：需要**
> **重点**：保存 `.java` 文件；使用 `javac` 编译；使用 `java` 运行；理解 `main` 入口。

## 先认识执行过程

编写的 `Hello.java` 是源码。`javac` 编译后生成 `Hello.class` 字节码；`java` 启动 JVM，加载并运行这个类。

```text
Hello.java  ──javac 编译──>  Hello.class  ──java 运行──>  控制台输出
```

## 完整例子

新建文本文件，保存为 **`Hello.java`**，内容如下。类前面的 `public` 表示公开类；文件名必须与这个公开类的名字 `Hello` 一致，大小写也要一致。

```java
import java.util.logging.Logger;

public class Hello {
    private static final Logger logger = Logger.getLogger(Hello.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：输出第一条消息
         * ========================================================================
         * 目标：验证 Java 程序可以运行。
         * 数据源：代码中的固定文本。
         * 操作：
         * 1) 在控制台输出文本。
         * 2) 记录步骤完成。
         */
        logger.info("开始运行第一个程序");
        // 1.1 输出固定文本
        System.out.println("Hello, Java!");
        logger.info("第一个程序运行完成");
    }
}
```

### 编译

在 PowerShell 中切换到 `Hello.java` 所在目录。确认当前目录后执行 `javac -encoding UTF-8 Hello.java`。若没有报错，目录中应出现 `Hello.class`。

如果使用较新的编译器，但实际运行时是 Java 11，可使用 `javac --release 11 -encoding UTF-8 Hello.java`，要求编译器支持这个目标版本。切勿只看 `javac -version`，还要查看 `java -version`。

### 运行

在同一目录执行 `java Hello`。标准输出应有 `Hello, Java!`；日志还会显示开始和完成信息，具体时间、格式和相对顺序可能不同。

注意运行参数是类名 `Hello`，不是 `Hello.class`。Java 11 起也支持单文件源码启动 `java Hello.java`；本节先分开编译和运行，便于理解源码与字节码的关系。

## 逐块理解

- `import java.util.logging.Logger` 引入 JDK 自带日志类；不需要安装第三方库。
- `public class Hello` 定义类。这个类中的大括号包住类成员。
- `public static void main(String[] args)` 是示例使用的程序入口。JVM 找到入口后从方法体开始执行；`args` 是命令行参数数组。
- `System.out.println(...)` 输出文本并换行；`logger.info(...)` 记录步骤日志。

大多数简单语句以分号结束，但类、方法和控制结构的花括号不是“每一行都要加分号”。

## 运行失败怎么查

1. 找不到 `javac`：检查 JDK 是否安装以及当前终端能否找到它。见 [[安装JDK]]。
2. 找不到主类：确认已在正确目录，编译产物存在，运行时类名和包名正确。
3. 版本不兼容：比较 `javac -version` 和 `java -version`；检查编译目标版本和运行时是否匹配。

## 手练与自测

把输出文本改为自己的练习内容，再编译运行一次。`javac` 负责哪一步？`java` 为什么接收 `Hello` 而不是 `Hello.class`？

参考：[廖雪峰：第一个Java程序](https://liaoxuefeng.com/books/java/quick-start/dev-env/first-program/index.html)；[Java 17 命令行工具](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html)。
