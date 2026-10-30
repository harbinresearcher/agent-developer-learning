# 使用 Iterator

> **重要程度：高**｜**手练：需要**
> **重点**：`hasNext`、`next`、迭代状态；遍历时用迭代器删除当前元素。

## 为什么需要迭代器

调用方经常只想逐个处理集合元素，不需要了解底层是数组还是其他结构。Iterator 提供统一的遍历方式：`hasNext()` 检查是否还有元素；`next()` 取出下一个元素并推进位置。

增强 for 遍历可迭代对象时也使用相应的迭代机制。它写起来简洁，但如果要在遍历过程中删除当前元素，需要明确所用集合和迭代器是否支持这种操作。

## 完整例子

保存为 `IteratorDemo.java`；执行 `javac -encoding UTF-8 IteratorDemo.java`，再执行 `java IteratorDemo`。

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;
import java.util.logging.Logger;

public class IteratorDemo {
    private static final Logger logger = Logger.getLogger(IteratorDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：遍历并移除不及格成绩
         * ========================================================================
         * 目标：使用迭代器安全移除当前元素。
         * 数据源：三条测试成绩。
         * 操作：
         * 1) 获取列表的迭代器。
         * 2) 遍历成绩并移除低于 60 的元素。
         */
        logger.info("开始筛选成绩");
        // 1.1 创建列表并获取迭代器
        List<Integer> scores = new ArrayList<>(Arrays.asList(80, 45, 90));
        Iterator<Integer> iterator = scores.iterator();
        // 1.2 读取当前成绩，必要时通过迭代器删除
        while (iterator.hasNext()) {
            Integer score = iterator.next();
            if (score < 60) {
                iterator.remove();
            }
        }
        System.out.println("保留成绩: " + scores);
        logger.info("成绩筛选完成");
    }
}
```

标准输出为 `保留成绩: [80, 90]`。

### 按时间顺序看

1. 取得列表的一个迭代器，迭代位置从第一个元素之前开始。
2. `hasNext()` 判断还有元素，`next()` 取出当前可用元素并推进迭代状态。
3. 当成绩为 45 时，通过**同一个迭代器**的 `remove()` 删除刚由 `next()` 返回的元素。
4. 循环继续读取 90。最后列表保留 80 和 90。

`Iterator.remove()` 是可选操作：部分迭代器不支持，会抛 `UnsupportedOperationException`。通常每次成功 `next()` 后才能针对该元素调用一次 remove；不能在 next 之前随意删除。

## 不要把快速失败当线程安全

在遍历一个普通集合时，通过集合自身方法做不兼容的结构性修改，可能触发 `ConcurrentModificationException`。这是一种尽力发现问题的机制，不应依赖它保证并发正确性。不同集合的迭代器可能有不同的一致性行为。

## 手练与自测

1. 将判断改成删除低于 90 的成绩，预测保留结果。
2. 为什么示例中用 `iterator.remove()`，而不是在循环里调用 `scores.remove(score)`？
3. 如果在第一次 `next()` 前调用 `iterator.remove()`，会怎样？

**答案要点**：① `[90]`。② 使用同一迭代器支持的删除流程，避免与遍历状态冲突。③ 通常抛 `IllegalStateException`；还需遵守具体迭代器契约。

参考：[教程：使用Iterator](https://liaoxuefeng.com/books/java/collection/iterator/index.html)；[Java Iterator API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Iterator.html)。
