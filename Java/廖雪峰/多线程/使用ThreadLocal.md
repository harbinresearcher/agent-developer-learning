# 使用ThreadLocal

> **重要程度：高**｜**手练：建议**
> **重点**：线程局部数据、线程池复用和清理。

## 核心概念

ThreadLocal 为每个线程提供独立的变量槽，可在同一线程的调用链中访问上下文。

## 工作中注意

线程池会复用线程，请求结束后需要 remove 等清理，避免上下文泄露；线程局部变量不自动跨异步线程传播。

## 手练

在线程池中设置请求标识，任务结束时清理；连续运行两个任务验证没有串值。

来源：[教程：使用ThreadLocal](https://liaoxuefeng.com/books/java/threading/thread-local/index.html)。
