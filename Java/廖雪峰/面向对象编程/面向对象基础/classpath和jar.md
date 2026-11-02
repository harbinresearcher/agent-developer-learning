# classpath 和 JAR

> **重要程度：高**｜**手练：需要**
> **重点**：包名与目录、类路径、JAR 入口、常见加载错误。

## 类路径是什么

JVM 按类的完整名称查找字节码。类路径（classpath）指定搜索目录和 JAR；例如 `demo.Hello` 对应类路径根目录下的 `demo/Hello.class`。根目录是 `demo` 的上一层，不是 `demo` 本身。

JAR 是 ZIP 格式的 Java 归档，可以包含类和资源。可执行 JAR 的清单可以声明入口类；普通依赖 JAR 不一定有入口，不能假定都能用 `java -jar` 启动。

## 完整练习

新建目录，在其中保存 `src/demo/Hello.java`：

```java
package demo;
import java.util.logging.Logger;
public class Hello {
    private static final Logger logger = Logger.getLogger(Hello.class.getName());
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：验证类路径和 JAR 入口
         * 数据源：当前程序的入口方法
         * 操作：1) 启动类 2) 输出可验证的消息
         * ======================================================================== */
        logger.info("开始运行示例");
        // 1.1 输出标准输出，供两种运行方式比较
        System.out.println("Hello JAR");
        logger.info("示例运行完成");
    }
}
```

使用 JDK 11 或更高版本，在新建目录中运行：

```powershell
javac -encoding UTF-8 -d out src/demo/Hello.java
java -cp out demo.Hello
jar --create --file hello.jar --main-class demo.Hello -C out .
java -jar hello.jar
```

两次运行的标准输出都是 `Hello JAR`，日志单独显示。`-d out` 把编译结果放到 `out/demo/Hello.class`；`-cp out` 指定类路径根；`--main-class` 写入 JAR 入口；`-C out .` 将 `out` 内的内容打包，而不是多包一层 `out`。

## 常见错误

- `ClassNotFoundException` 或找不到主类：检查完整类名、包声明、输出目录和当前工作目录。`demo.Hello` 用点分隔，不写 `.class` 后缀。
- `NoClassDefFoundError` 可能表示运行时所需类不可用，也可能是类初始化失败后的后续表现；应查看完整异常与最早的原因。
- `no main manifest attribute` 表示 JAR 清单没有可用入口；依赖库通常不应直接运行。
- Windows 的多个类路径项用分号分隔，Unix 类系统通常用冒号；包含空格的路径需按当前 shell 的规则引用。

## 工作中怎么用

项目通常用 Maven 或 Gradle 管理依赖和构建，不把开发机绝对路径硬编码到源码。普通 `java -jar` 的依赖查找方式与 Spring Boot 可执行 JAR 的嵌套依赖机制不同，不能假定普通 JAR 会自动包含所有依赖。模块路径还有单独的可读性和导出规则。

## 动手与自查

1. 运行四条命令，确认目录运行和 JAR 运行输出相同。
2. 将类路径错误地设为 `out/demo`，解释为什么 JVM 找不到 `demo.Hello`。
3. 去掉 `--main-class` 创建另一个 JAR，观察 `java -jar` 的错误。

来源：[教程：classpath 和 jar](https://liaoxuefeng.com/books/java/oop/basic/classpath-jar/index.html)、[Java 启动器](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html)。
