# 编写JUnit测试

> **重要程度：高**｜**手练：需要**
> **重点**：测试结构、断言、正常值与边界。

## 解决什么问题

单元测试自动检查一小段代码的可观察行为。一次测试通常准备输入、调用目标并断言输出；失败时应该能看出哪个规则被破坏，而不是只打印“运行成功”。

## 完整例子

保存为 `CalculatorTest.java`。这里把简单被测方法放在测试类里便于独立运行；实际项目通常把生产代码与测试代码放在各自目录。

```java
import org.junit.jupiter.api.Test;
import java.util.logging.Logger;
import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {
    private static final Logger logger = Logger.getLogger(CalculatorTest.class.getName());
    @Test
    void addsPositiveNumbers() {
        /*
         * ========================================================================
         * 步骤1：验证两个正整数相加
         * ========================================================================
         * 数据源：两个固定整数
         * 操作：
         * 1) 调用加法方法
         * 2) 断言返回值
         */
        logger.info("开始测试正数相加");
        // 1.1 调用并检查可观察结果
        assertEquals(5, add(2, 3));
        logger.info("正数相加测试完成");
    }
    @Test
    void addsZero() {
        /*
         * ========================================================================
         * 步骤1：验证零值边界
         * ========================================================================
         * 数据源：零和一个整数
         * 操作：
         * 1) 执行加法
         * 2) 检查边界结果
         */
        logger.info("开始测试零值边界");
        // 1.1 断言 0 不改变另一个加数
        assertEquals(7, add(0, 7));
        logger.info("零值边界测试完成");
    }
    static int add(int a, int b) { return a + b; }
}
```

## 运行方式

本例使用 JUnit Platform Console Standalone 1.14.4（包含 JUnit Jupiter），可在 Java 11 运行。将 [standalone jar](https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-console-standalone/1.14.4/junit-platform-console-standalone-1.14.4.jar) 放在源码所在目录，然后运行：

```text
javac -encoding UTF-8 -cp junit-platform-console-standalone-1.14.4.jar CalculatorTest.java
java -jar junit-platform-console-standalone-1.14.4.jar execute --class-path . --select-class CalculatorTest --details summary --disable-ansi-colors
```

本示例已用 Java 11 和上述 jar 验证。JUnit 5 系列可在 Java 11 上运行；JUnit 6 要求 Java 17 或更高，项目应按 JDK 和构建配置选兼容版本。日志写到标准错误，测试报告格式可能因版本而异。

默认运行结果：2 个测试成功、0 个跳过、0 个失败。

## 执行过程

测试发现器找到两个 `@Test` 方法。第一个检查 `add(2, 3)` 为 5，第二个检查零值边界。`assertEquals(expected, actual)` 在不相等时让当前测试失败；测试通过不代表所有输入、整数溢出或其他功能都已覆盖。

JUnit Jupiter 通常为每个测试方法创建独立测试类实例；不要依赖测试方法的执行顺序。普通测试不应依赖真实网络、当前时间或另一条测试留下的数据；需要跨组件验证时写明确的集成测试。

## 工作中怎么用

将生产代码放在 `src/main/java`，测试放在 `src/test/java`，让 Maven 或 Gradle 管理 JUnit 依赖并在构建中执行。选择正常、边界和失败路径，检查可观察行为；测试通过只证明这些断言通过。

## 手练与答案

把加法方法改为 `a - b`：正数用例应失败。把它恢复后重新运行，两条测试应通过；零值用例单独通过不能证明正数逻辑正确。

参考：[廖雪峰：编写JUnit测试](https://liaoxuefeng.com/books/java/unit-test/junit-test/index.html)；[JUnit 5 用户指南](https://docs.junit.org/5.14.4/overview.html)。
