# 安装与检查 JDK

> **重要程度：高**｜**手练：需要**
> **重点**：区分 JDK 和 JVM；检查 `java` 与 `javac`；理解系统默认版本和项目版本。

## 需要哪些东西

JDK 是 Java 开发工具包，包含 `javac` 编译器及运行 Java 所需工具；JVM 负责执行 Java 字节码。学习和开发 Java 程序通常需要 JDK，而不只是能够运行已有程序的环境。

JDK 有多个发行版和版本。选择前看项目要求、依赖兼容性和发行版授权；团队项目优先按仓库说明配置。教程示例使用的版本不一定就是当前项目版本。

## Windows 上先检查，再决定是否安装

打开 PowerShell，分别运行：

```powershell
java -version
javac -version
where.exe java
where.exe javac
```

前两条查看运行时与编译器版本；后两条帮助查看命令会从哪些位置找到可执行文件。命令找不到，不一定说明机器完全没有安装：也可能是当前终端的 PATH 没有指向它。

**两者可能来自不同 JDK。** 编译器版本较新、运行时较旧时，默认编译产物可能无法运行。也要检查 IDE 的项目 JDK，因为它可能与终端默认值不同。

本次整理时观察到 `java` 为 11、`javac` 为 17；这是当时的环境快照，不保证你现在仍是这个状态。请以重新运行命令的结果为准。

## 如何验证安装

按发行版官方说明安装合适的 JDK 后，重新打开终端，再运行上述版本命令。然后完成 [[第一个Java程序]] 的编译与运行。版本号可见只是第一步，实际编译运行成功才能证明该示例在当前终端可用。

如果项目要求较旧目标版本，编译器可能支持 `javac --release` 指定目标版本；仍需确认项目依赖及运行时兼容，不能靠一个参数解决所有版本问题。

## 自测

1. `java -version` 和 `javac -version` 分别检查什么？
2. 编译器 17、运行时 11，为什么可能出现版本错误？
3. 终端和 IDE 使用的 JDK 为什么需要分别确认？

参考：[廖雪峰：安装JDK](https://liaoxuefeng.com/books/java/quick-start/dev-env/install-jdk/index.html)；[Oracle JDK 文档](https://docs.oracle.com/en/java/javase/)。
