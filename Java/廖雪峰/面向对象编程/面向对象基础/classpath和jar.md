# classpath 与 JAR

> **重要程度：中**｜**手练：建议**
> **重点**：类路径、完整类名、JAR 用途与常见加载错误。

## 类路径是什么

JVM 需要根据类的完整名称找到字节码。classpath 指定搜索位置，可以包含目录和 JAR；目录结构应与包名对应。例如完整类名 `com.example.Main` 对应目录下的 `com/example/Main.class`。

JAR 是 Java 归档文件，可以打包类和资源。有些 JAR 可通过清单中的入口配置启动，但并非每个 JAR 都能直接用 `java -jar` 运行。项目依赖和 classpath 通常由 Maven、Gradle 或 IDE 管理，不需要手工拼接长路径。

## 排错

`ClassNotFoundException` 常见于按名称显式加载类但找不到；`NoClassDefFoundError` 可能与运行期类不可用或类初始化失败有关，需看完整异常和原因，不能只凭错误名断定。核对运行目录、依赖是否进入运行时、包名和版本。

## 手练

将两个带包名的类编译到输出目录，确认目录层级；故意指定错误的运行类路径，观察错误后修复。

来源：[教程：classpath 和 jar](https://liaoxuefeng.com/books/java/oop/basic/classpath-jar/index.html)。
