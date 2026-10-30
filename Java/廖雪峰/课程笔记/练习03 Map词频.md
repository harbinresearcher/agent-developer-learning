# 手练：Map 统计词频

> **重要程度：高**｜**手练：需要**
> **重点**：遍历、Map 的键值含义、默认值与相等判断。

将代码保存为 `WordCountDemo.java`。先运行 `javac -encoding UTF-8 WordCountDemo.java`，再运行 `java WordCountDemo`。

```java
import java.util.Map;
import java.util.TreeMap;
import java.util.logging.Logger;

public class WordCountDemo {
    private static final Logger logger = Logger.getLogger(WordCountDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：统计并输出词频
         * ========================================================================
         * 目标：按词语统计出现次数。
         * 数据源：本地字符串数组。
         * 操作：
         * 1) 遍历数组并累加计数。
         * 2) 按键的自然顺序输出结果。
         */
        logger.info("开始统计词频");
        // 1.1 准备数据和映射
        String[] words = {"java", "python", "java"};
        Map<String, Integer> counts = new TreeMap<>();
        // 1.2 对每个单词累加计数
        for (String word : words) {
            counts.put(word, counts.getOrDefault(word, 0) + 1);
        }
        // 1.3 输出每个键值对
        for (Map.Entry<String, Integer> entry : counts.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
        logger.info("词频统计完成");
    }
}
```

不存在的词通过 `getOrDefault(word, 0)` 取到初始计数。本例用 `TreeMap` 让输出按键排序；换成 `HashMap` 后不要依赖输出顺序。

手练：新增一个重复词并预测结果；再换成 `HashMap`，解释为什么测试不应依赖遍历顺序。
