# 使用 Map

> **重要程度：高**｜**手练：需要**
> **重点**：键值映射、`put` / `get`、缺失键处理、键的相等规则。

## 先看需求

要统计每个单词出现几次。用列表只知道单词顺序，还得自己查它对应哪个计数。Map 用**键**找对应的**值**：这里单词是键，出现次数是值。

`Map<String, Integer>` 表示按字符串键、整数值使用的映射；`HashMap` 是常见实现。它不保证遍历顺序。

## 完整例子

保存为 `MapDemo.java`；运行 `javac -encoding UTF-8 MapDemo.java`，再运行 `java MapDemo`。
```java
import java.util.HashMap;
import java.util.Map;
import java.util.logging.Logger;

public class MapDemo {
    private static final Logger logger = Logger.getLogger(MapDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：统计单词出现次数
         * ========================================================================
         * 目标：用单词作为键、次数作为值。
         * 数据源：三条测试单词。
         * 操作：
         * 1) 根据现有次数更新映射。
         * 2) 查询已有键和缺失键。
         */
        logger.info("开始统计单词");
        // 1.1 遍历单词并累计次数
        String[] words = {"java", "python", "java"};
        Map<String, Integer> counts = new HashMap<>();
        for (String word : words) {
            counts.put(word, counts.getOrDefault(word, 0) + 1);
        }
        // 1.2 查询已有键与缺失键
        System.out.println("java: " + counts.get("java"));
        System.out.println("包含 go: " + counts.containsKey("go"));
        logger.info("单词统计完成");
    }
}
```

标准输出为 `java: 2` 和 `包含 go: false`。

### 按执行顺序看

1. 创建空 `HashMap`，依次读到 `java`、`python`、`java`。
2. `getOrDefault(word, 0)` 在键不存在时使用 0；第一次遇到 `java` 就把它更新为 1。
3. 第二次遇到 `java` 时，原值为 1，更新后变为 2。`put` 对已有键更新其对应值，不会创建两个同样的键。
4. `containsKey("go")` 检查键是否存在，本例是 false。

## 边界和工作中注意

`get(key)` 返回 null 可能表示键不存在；在允许 null 值的实现里，也可能表示键存在但值是 null。需要区分时使用 `containsKey`。不同 Map 实现对 null 和顺序的约定不同，不能套用 HashMap 的规则到所有实现。

作为 HashMap 键的对象要正确实现 `equals` 和 `hashCode`；修改参与相等判断的键字段可能使后续查找异常。见 [[编写equals方法]] 与 [[编写hashCode方法]]。

本例的“读取旧值再 put”适合单线程演示；多个线程同时更新同一个普通 HashMap 时，复合操作不能当作原子操作。

## 手练与自测

1. 加入第三个 `java`，预测最终次数。
2. `get("go")` 和 `containsKey("go")` 分别回答什么问题？
3. 为什么测试 HashMap 内容时不应依赖输出顺序？

**答案要点**：① 3。② 前者读取对应值，后者检查是否存在该键。③ HashMap 不承诺迭代顺序。

参考：[教程：使用Map](https://liaoxuefeng.com/books/java/collection/map/index.html)；[Java Map API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Map.html)。
