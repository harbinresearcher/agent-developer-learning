# 使用ReentrantLock

> **重要程度：中**｜**手练：建议**
> **重点**：显式获取、finally 释放和可中断。

## 核心概念

ReentrantLock 提供显式锁控制，可以按 API 使用可中断获取、尝试获取和 Condition；同一线程可重入。

## 工作中注意

成功获取后应在 finally 中释放，避免异常导致锁泄漏；不要把手动锁视作总比 synchronized 快。

## 手练

保护共享计数，确保更新抛异常时仍释放锁，并让另一个线程继续执行。

来源：[教程：使用ReentrantLock](https://liaoxuefeng.com/books/java/threading/synchronize/reentrant/index.html)。
