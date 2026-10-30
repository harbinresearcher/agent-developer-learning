# 使用CompletableFuture

> **重要程度：高**｜**手练：建议**
> **重点**：组合、异常传播和执行器。

## 核心概念

CompletableFuture 可连接多个异步阶段，例如映射结果、组合任务和处理异常。同步与异步版本的方法在线程选择上可能不同。

## 工作中注意

指定适合任务的执行器并设置超时；不要忽略异常，也不要把阻塞 IO 无限制放入共享线程池。

## 手练

组合两个独立结果，再模拟一个阶段失败，检查错误如何传给最终调用方。

来源：[教程：使用CompletableFuture](https://liaoxuefeng.com/books/java/threading/completable-future/index.html)。
