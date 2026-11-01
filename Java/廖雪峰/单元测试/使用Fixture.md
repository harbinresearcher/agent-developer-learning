# 使用Fixture

> **重要程度：高**｜**手练：需要**
> **重点**：每次测试准备与清理、测试隔离。

## 解决什么问题

Fixture 是测试所需的初始环境和数据。JUnit Jupiter 的 `@BeforeEach` 在每个测试前运行，适合准备互不影响的状态；`@AfterEach` 可用于清理由测试创建且需要关闭的资源。

## 完整例子

保存为 `FixtureTest.java`。

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import java.util.ArrayList;
import java.util.List;
import java.util.logging.Logger;
import static org.junit.jupiter.api.Assertions.assertEquals;

class FixtureTest {
    private static final Logger logger = Logger.getLogger(FixtureTest.class.getName());
    private List<String> items;
    @BeforeEach
    void setUp() {
        /*
         * ========================================================================
         * 步骤1：准备独立列表
         * ========================================================================
         * 数据源：新建的内存列表
         * 操作：
         * 1) 创建空列表
         * 2) 为当前测试提供初始状态
         */
        logger.info("开始准备测试数据");
        // 1.1 每个测试前重新创建列表
        items = new ArrayList<>();
        logger.info("测试数据准备完成");
    }
    @Test
    void startsEmpty() {
        /*
         * ========================================================================
         * 步骤1：检查初始状态
         * ========================================================================
         * 数据源：当前测试的列表
         * 操作：
         * 1) 读取元素数量
         * 2) 检查空列表
         */
        logger.info("开始检查初始状态");
        // 1.1 检查初始大小
        assertEquals(0, items.size());
        logger.info("初始状态检查完成");
    }
    @Test
    void addOneItem() {
        /*
         * ========================================================================
         * 步骤1：检查添加行为
         * ========================================================================
         * 数据源：当前测试的空列表
         * 操作：
         * 1) 添加一个元素
         * 2) 检查列表大小
         */
        logger.info("开始检查添加行为");
        // 1.1 修改并检查列表
        items.add("A");
        assertEquals(1, items.size());
        logger.info("添加行为检查完成");
    }
}
```

## 运行方式

本例使用 JUnit Platform Console Standalone 1.14.4（包含 JUnit Jupiter），可在 Java 11 运行。将 [standalone jar](https://repo.maven.apache.org/maven2/org/junit/platform/junit-platform-console-standalone/1.14.4/junit-platform-console-standalone-1.14.4.jar) 放在源码所在目录，然后运行：

```text
javac -encoding UTF-8 -cp junit-platform-console-standalone-1.14.4.jar FixtureTest.java
java -jar junit-platform-console-standalone-1.14.4.jar execute --class-path . --select-class FixtureTest --details summary --disable-ansi-colors
```

本示例已用 Java 11 和上述 jar 验证。JUnit 5 系列可在 Java 11 上运行；JUnit 6 要求 Java 17 或更高，项目应按 JDK 和构建配置选兼容版本。日志写到标准错误，测试报告格式可能因版本而异。

默认运行结果：2 个测试成功、0 个跳过、0 个失败。

## 执行过程

每个 `@Test` 前都会运行 `setUp()`，创建新的空列表。`startsEmpty` 检查初始大小，`addOneItem` 添加一项后检查大小。两条测试各自得到自己的初始列表，不应依赖谁先运行。

默认生命周期通常是每个测试方法一个测试类实例；即使修改了生命周期，也仍应为测试显式维护隔离。`@BeforeAll`、`@AfterAll` 用于整个类范围的准备与清理；默认生命周期下这两个方法通常必须是 static，使用 PER_CLASS 时规则不同。

## 工作中怎么用

测试只清理它自己创建的文件、数据或连接。若准备步骤失败，相关测试可能无法执行；不要把耗时外部系统强行放进所有单元测试。涉及数据库等共享资源时，明确事务、并发与清理策略。

## 手练与答案

删除 `@BeforeEach`：两个测试中 `items` 未被初始化，访问会抛 `NullPointerException`。每个测试都重新创建列表，因此一个测试添加元素不会成为另一个测试的前置条件。

参考：[廖雪峰：使用Fixture](https://liaoxuefeng.com/books/java/unit-test/fixture/index.html)；[JUnit 5 用户指南](https://docs.junit.org/5.14.4/overview.html)。
