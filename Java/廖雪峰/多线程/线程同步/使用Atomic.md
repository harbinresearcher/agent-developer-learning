# 使用Atomic

> **重要程度：高**｜**手练：建议**
> **重点**：原子变量、CAS 和复合不变量。

## 核心概念

AtomicInteger 等提供对单个变量的原子操作，常用于计数、状态更新等。

## 工作中注意

原子变量不能自动保护多个字段之间的不变量；高竞争或复杂逻辑需要测量并评估锁或其他方案。

## 手练

多线程累加同一个 AtomicInteger 并核对结果，再说明两个变量同时更新为何仍需额外设计。

来源：[教程：使用Atomic](https://liaoxuefeng.com/books/java/threading/synchronize/atomic/index.html)。
