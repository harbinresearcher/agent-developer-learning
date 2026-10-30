# 使用Queue

> **重要程度：中**｜**手练：建议**
> **重点**：队列、空队列和方法差异。

## 核心概念

Queue 通常按队列规则提供添加、查看和移除；`offer`、`poll`、`peek` 与部分抛异常的方法在失败或空队列时的行为不同。

## 工作中注意

队列是否允许 null、是否有容量以及是否线程安全由具体实现决定；跨线程生产消费可考虑并发包中的阻塞队列。

## 手练

把三个任务放入队列并依次处理；空队列时比较 poll 和 remove 的行为。

来源：[教程：使用Queue](https://liaoxuefeng.com/books/java/collection/queue/index.html)。
