# 使用Set

> **重要程度：高**｜**手练：需要**
> **重点**：唯一性与不同实现。

## 核心概念

Set 表达不包含重复元素的集合。HashSet 使用相等和哈希规则判断，TreeSet 使用排序比较规则；不同实现的遍历顺序不同。

## 工作中注意

Set 不会自动让其元素不可变。修改参与相等判断的字段可能破坏哈希集合查找，选元素类型时应注意。

## 手练

将重复标签加入 Set，检查数量；比较 HashSet 和 TreeSet 的顺序约束。

来源：[教程：使用Set](https://liaoxuefeng.com/books/java/collection/set/index.html)。
