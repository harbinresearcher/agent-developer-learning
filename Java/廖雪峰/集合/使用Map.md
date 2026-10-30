# 使用Map

> **重要程度：高**｜**手练：需要**
> **重点**：键值映射、put、get 与缺失键。

## 核心概念

Map 将键映射到值；`put` 写入，`get` 查询。不同实现对顺序、null 和并发的约束不同。`HashMap` 不保证迭代顺序。

## 工作中注意

`get` 返回 null 可能是键不存在，也可能是键存在而值为 null；需要区分时使用 `containsKey`。不要在多线程下把普通 HashMap 的复合操作视为安全。

## 手练

使用 Map 统计词频，结合 [[练习03 Map词频]] 理解默认计数。

来源：[教程：使用Map](https://liaoxuefeng.com/books/java/collection/map/index.html)。
