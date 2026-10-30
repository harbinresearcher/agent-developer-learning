# 使用Collections

> **重要程度：中**｜**手练：建议**
> **重点**：集合工具方法、视图和不可变性。

## 核心概念

`Collections` 是集合工具类，提供排序、反转、空集合、同步包装和不可修改包装等工具；不要与 Collection 接口混淆。

## 工作中注意

不可修改包装常是底层集合的视图，底层变化仍可能可见；同步包装也不意味着复合操作和遍历可以省略外部同步。

## 手练

创建原列表和不可修改视图，修改原列表并观察视图；解释它与独立快照的区别。

来源：[教程：使用Collections](https://liaoxuefeng.com/books/java/collection/collections/index.html)。
