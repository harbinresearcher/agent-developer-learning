# 使用Concurrent集合

> **重要程度：高**｜**手练：建议**
> **重点**：并发映射、队列与复合操作。

## 核心概念

并发集合为特定操作提供线程安全保证，例如 ConcurrentHashMap 和阻塞队列；不同集合的迭代和一致性语义不同。

## 工作中注意

线程安全的单次操作不自动让多步业务逻辑原子；需要原子更新时选用合适 API 或额外同步。

## 手练

让多个线程统计词频，比较不安全的复合更新与集合提供的原子更新方法。

来源：[教程：使用Concurrent集合](https://liaoxuefeng.com/books/java/threading/synchronize/concurrent/index.html)。
