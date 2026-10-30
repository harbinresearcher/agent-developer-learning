# Java 入门与开发环境

> **重要程度：高**｜**手练：需要**
> **重点**：区分 JDK、JVM；在项目中确认编译器和运行时版本；能编译运行 Java 程序。

## 从源码到运行

Java 源码通常保存在 `.java` 文件中。`javac` 将源码编译为 `.class` 字节码，`java` 启动 JVM 执行程序。JDK 是开发工具包，包含编译器和运行工具；JVM 是运行字节码的虚拟机。

`java -version` 查看运行时，`javac -version` 查看编译器。两条命令可能指向不同的安装目录；编译版本高于运行时支持版本时，程序可能报 `UnsupportedClassVersionError`。IDE 里的项目 JDK 也要单独检查。

项目中使用团队指定的 JDK 版本。本文大多数基础示例按 Java 11 或更低语法编写；记录类、虚拟线程等新特性会单独标明版本。

## 第一个程序

先阅读 [[面向对象编程/面向对象基础/构造方法|构造方法]] 的完整示例：保存为 `Main.java`，在文件所在目录运行 `javac -encoding UTF-8 Main.java`，再运行 `java Main`。入口是 `public static void main(String[] args)`；公开类名与源文件名必须匹配。

## 手练

分别运行 `java -version` 和 `javac -version`；编译并运行上面的示例。检查两个版本是否一致，说明为什么运行时较旧可能无法加载新字节码。

来源：[教程：搭建开发环境](https://liaoxuefeng.com/books/java/quick-start/dev-env/index.html)；[Java 语言规范](https://docs.oracle.com/javase/specs/)。
