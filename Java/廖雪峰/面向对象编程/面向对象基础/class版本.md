# class 版本

> **重要程度：中**｜**手练：建议**
> **重点**：编译目标版本、运行时版本与版本不兼容的排查。

Java 编译器生成的 `.class` 文件有版本信息。较旧的 JVM 不一定能运行由较新版本生成的字节码；常见表现是 `UnsupportedClassVersionError`。即使字节码版本可加载，依赖的库或 API 也必须在目标运行环境中存在。

检查 `javac -version` 和 `java -version`，再检查 IDE、构建工具及实际部署环境使用的 JDK。两条命令可能来自不同安装位置。本机当前工作中曾发现编译器与运行时版本不同，环境以后可能变化，应重新检查。

交叉编译时可按项目要求设置 `javac --release` 或构建工具的对应选项；它同时约束目标平台的语言和标准 API。示例需要的新特性超过目标版本时，不能只改参数让旧运行时理解新特性。

## 手练

记录两个版本命令的输出；使用项目允许的目标版本编译一个简单程序，再用目标运行时运行。出现不兼容时，确认编译目标、依赖和运行时三者是否匹配。

来源：[教程：class版本](https://liaoxuefeng.com/books/java/oop/basic/class-version/index.html)；[javac 选项](https://docs.oracle.com/en/java/javase/17/docs/specs/man/javac.html)。
