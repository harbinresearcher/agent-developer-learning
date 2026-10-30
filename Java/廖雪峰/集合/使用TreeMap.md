# 使用TreeMap

> **重要程度：中**｜**手练：建议**
> **重点**：排序、Comparator 与键相等。

## 核心概念

TreeMap 按键的自然顺序或提供的 Comparator 排序；查找和更新的语义依赖比较结果。

## 工作中注意

比较器若对不同对象返回零，TreeMap 会将它们视为同一排序位置的键；要确保比较规则满足业务需求，并与 equals 的预期保持一致。

## 手练

按时间或名称对记录排序；构造两个比较结果为零的不同键，观察覆盖行为。

来源：[教程：使用TreeMap](https://liaoxuefeng.com/books/java/collection/tree-map/index.html)。
