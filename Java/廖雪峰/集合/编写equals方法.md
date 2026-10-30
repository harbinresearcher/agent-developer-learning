# 编写equals方法

> **重要程度：高**｜**手练：需要**
> **重点**：值相等与相等契约。

## 核心概念

`equals` 表达对象的逻辑相等；两个不同实例可能表示相同的业务值。重写时要满足自反、对称、传递、一致及对 null 返回 false 等契约。

## 工作中注意

相等字段应与业务身份一致；重写 equals 时必须同时考虑 hashCode，否则在哈希集合中查找可能违反预期。

## 手练

编写表示坐标的值对象，比较相同和不同坐标，并测试与 null 的比较。

来源：[教程：编写equals方法](https://liaoxuefeng.com/books/java/collection/equals/index.html)。
