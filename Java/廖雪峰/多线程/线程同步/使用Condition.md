# 使用Condition

> **重要程度：中**｜**手练：按需**
> **重点**：条件等待和关联锁。

## 核心概念

Condition 与特定 Lock 配合，为线程提供 await 和 signal 等条件等待机制；可以为不同条件建立独立等待队列。

## 工作中注意

调用 await 或 signal 前需持有关联锁；用循环检查条件并处理 InterruptedException。

## 手练

为有界队列区分“非空”和“未满”条件，说明两类线程分别等待什么。

来源：[教程：使用Condition](https://liaoxuefeng.com/books/java/threading/synchronize/condition/index.html)。
