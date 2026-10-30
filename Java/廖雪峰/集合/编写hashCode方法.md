# 编写hashCode方法

> **重要程度：高**｜**手练：需要**
> **重点**：equals 与 hashCode 的契约。

## 核心概念

如果两个对象通过 equals 相等，它们的 hashCode 必须相等；hashCode 相等并不保证 equals 相等。哈希容器先借助哈希定位，再按相等规则判断。

## 工作中注意

不要在对象作为 HashMap 键或 HashSet 元素期间修改参与 equals 和 hashCode 的字段，否则可能无法按预期找到它。

## 手练

为坐标对象同时实现 equals 和 hashCode，放入 HashSet 后用相同值的新对象查找。

来源：[教程：编写hashCode方法](https://liaoxuefeng.com/books/java/collection/hashcode/index.html)。
