# 使用Stack

> **重要程度：低**｜**手练：按需**
> **重点**：旧 Stack 与 Deque 的关系。

## 核心概念

Stack 表达后进先出，但它是较早的类并继承 Vector。新代码实现普通栈时通常优先考虑 Deque 接口和适合的实现。

## 工作中注意

使用前确认是否需要线程安全；多线程场景的正确性不能只靠替换一个容器名称解决。

## 手练

使用 Deque 的 push、peek、pop 完成括号匹配练习。

来源：[教程：使用Stack](https://liaoxuefeng.com/books/java/collection/stack/index.html)。
