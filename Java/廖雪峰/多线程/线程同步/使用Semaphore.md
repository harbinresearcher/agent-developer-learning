# 使用Semaphore

> **重要程度：中**｜**手练：建议**
> **重点**：许可数量与并发限流。

## 核心概念

Semaphore 控制同时持有许可的任务数量，常用于限制对稀缺资源的并发访问。

## 工作中注意

成功获取许可后必须在 finally 释放；许可数量应与下游容量相匹配，限流不自动等于速率限制。

## 手练

用少量许可模拟有限连接，记录任意时刻最大并发数，验证异常时许可也会释放。

来源：[教程：使用Semaphore](https://liaoxuefeng.com/books/java/threading/synchronize/semaphore/index.html)。
