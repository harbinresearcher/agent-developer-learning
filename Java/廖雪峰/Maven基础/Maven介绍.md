# Maven 介绍

> **重要程度：高**｜**手练：需要**
> **重点**：项目坐标、标准目录、依赖、构建生命周期与测试产物。

## 解决什么问题

Java 项目不仅要编译源码，还要管理第三方库、运行测试和生成 JAR。Maven 使用 `pom.xml` 描述项目、依赖与构建配置，并通过插件完成具体任务。团队成员使用同一项目配置，仍需确保 JDK、Maven、仓库和环境兼容。

## 完整例子：可编译、测试并打包的项目

确认已有 JDK 和 Maven：运行 `java -version`、`javac -version`、`mvn -version`；注意 Maven 输出的 Java 版本决定它实际使用哪个 JDK。以下例子用 JDK 17 运行 Maven，编译目标为 Java 11。建立目录：

```text
calculator/
├── pom.xml
└── src/
    ├── main/java/demo/Calculator.java
    └── test/java/demo/CalculatorTest.java
```

`pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>example.notes</groupId>
    <artifactId>calculator</artifactId>
    <version>1.0.0</version>
    <properties>
        <maven.compiler.release>11</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.14.4</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build><plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.14.1</version>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.5.4</version>
        </plugin>
    </plugins></build>
</project>
```

`src/main/java/demo/Calculator.java`：

```java
package demo;
import java.util.logging.Logger;
public class Calculator {
    private static final Logger logger = Logger.getLogger(Calculator.class.getName());
    public int add(int first, int second) {
        /*
         * ========================================================================
         * 步骤1：计算整数和
         * ========================================================================
         * 数据源：调用方提供的两个整数
         * 操作：
         * 1) 计算整数和
         * 2) 返回计算结果
         */
        logger.info("开始计算整数和");
        // 1.1 使用精确加法检查溢出
        int result = Math.addExact(first, second);
        logger.info("整数和计算完成");
        return result;
    }
}
```

`src/test/java/demo/CalculatorTest.java`：

```java
package demo;
import java.util.logging.Logger;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
class CalculatorTest {
    private static final Logger logger = Logger.getLogger(CalculatorTest.class.getName());
    @Test
    void addsTwoNumbers() {
        /*
         * ========================================================================
         * 步骤1：验证整数加法
         * ========================================================================
         * 数据源：两个固定整数
         * 操作：
         * 1) 创建计算器并执行加法
         * 2) 检查返回值
         */
        logger.info("开始测试整数加法");
        // 1.1 断言计算结果
        assertEquals(5, new Calculator().add(2, 3));
        logger.info("整数加法测试完成");
    }
}
```

在包含 pom.xml 的 `calculator` 目录运行 `mvn clean verify`。首次运行可能下载依赖和插件，需要可用的仓库。验证结果应包含 1 条测试成功、0 失败，产物为 `target/calculator-1.0.0.jar`。本例已用 Maven 3.9.16、JDK 17 和 Java 11 编译目标验证；这个 JAR 是库示例，没有配置可直接 `java -jar` 启动的入口。

## 按顺序理解

`groupId`、`artifactId`、`version` 共同标识制品；此例为 `example.notes:calculator:1.0.0`。`src/main/java` 放生产代码，`src/test/java` 放测试；JUnit 的 `test` 依赖供测试阶段使用。`maven.compiler.release` 指定编译目标，并不自动帮你安装对应 JDK。

`clean` 清理项目构建输出；`verify` 按生命周期执行其之前需要的阶段，通常包括编译、测试和打包，再执行项目绑定的验证。插件版本在 pom 中明确，减少不同环境下的意外变化；成功仅证明实际运行的构建检查通过，不等于生产部署成功。

## 工作中怎么用

优先按项目 README 和 Maven Wrapper 使用项目约定版本；检查依赖树、编译目标、仓库与测试报告。首次构建失败时定位第一处实际错误，区分源码、测试、依赖下载和 JDK 不匹配。不要把仓库凭据写入 pom 或提交到版本库。

## 手练与答案

1. 把测试的预期值从 5 改成 6，再运行 `mvn test`：该测试应失败，构建不能视作通过；恢复后重跑。
2. `mvn package` 会经历此前的相关生命周期阶段；`mvn install` 还会安装到本地仓库，但不等于发布到远程。

参考：[廖雪峰：Maven介绍](https://liaoxuefeng.com/books/java/maven/basic/index.html)；[Apache Maven 生命周期](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)。
