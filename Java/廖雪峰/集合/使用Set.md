# 使用 Set

> **重要程度：高**｜**手练：需要**
> **重点**：用集合表达唯一性；理解 Set 怎样判断重复；区分 HashSet 和 TreeSet 的顺序。

## 先看问题：名单如何去重

假设标签依次是 `Java`、`Python`、`Java`，希望只保留不同的标签。Set 表达的是“不包含重复元素”的集合；加入已存在的元素，不会增加集合的元素数量。

与 List 不同，Set 的主要用途不是按下标访问。具体实现有不同规则：`HashSet` 不承诺遍历顺序；`LinkedHashSet` 通常按插入顺序遍历；`TreeSet` 按自然顺序或提供的比较器排序。

## 对象重复是怎么判断的

先运行 [[编写equals方法]] 的 `EqualityDemo.java`：两个独立的 `Point(1, 2)` 被加入 `HashSet`，最终数量为 1。原因是 Point 定义了与坐标一致的 `equals` 和 `hashCode`，集合按哈希及相等契约识别重复。

关键区别：两个不同对象可以逻辑相等；`==` 为 false 并不意味着它们在 HashSet 中一定会被当成不同值。对于 `TreeSet`，排序比较器的比较结果会影响它如何区分元素；比较器返回 0 的不同对象可能被视为同一排序位置。

## 工作中注意

作为 HashSet 元素的对象，如果参与 `equals` 和 `hashCode` 的字段在加入后改变，之后可能无法按预期查找或删除。优先使用不可变值，或保证相关字段在集合中保持稳定。

不同 Set 实现对 null、排序和线程安全的规则不同；选择具体实现前查 API，不要假设所有 Set 都一样。

## 手练与自测

1. 将 `Java`、`Python`、`Java` 依次加入 HashSet，数量是多少？
2. 为什么不能依赖 HashSet 的遍历顺序？
3. 为什么自定义元素要同时考虑 equals 和 hashCode？

**答案要点**：① 2。② HashSet 不承诺该顺序。③ 相等对象必须有相同哈希，哈希集合依赖这个契约。

参考：[教程：使用Set](https://liaoxuefeng.com/books/java/collection/set/index.html)；[Java Set API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Set.html)。
