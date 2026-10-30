# 使用Deque

> **重要程度：中**｜**手练：建议**
> **重点**：双端操作、队列与栈。

## 核心概念

Deque 是双端队列，可以从两端添加、查看和删除元素；`ArrayDeque` 常用于单线程队列或栈。

## 工作中注意

选择对应的操作对，如队列的 offerLast/pollFirst 或栈的 push/pop；不要把普通 ArrayDeque 当作线程安全结构。

## 手练

用 Deque 先实现先进先出队列，再实现后进先出栈，比较输出顺序。

来源：[教程：使用Deque](https://liaoxuefeng.com/books/java/collection/deque/index.html)。
