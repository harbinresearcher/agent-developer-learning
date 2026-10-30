# 使用wait和notify

> **重要程度：中**｜**手练：建议**
> **重点**：监视器、条件循环和虚假唤醒。

## 核心概念

wait、notify 等必须在持有对应对象监视器的前提下使用；wait 会释放该监视器并等待，唤醒后重新竞争锁。

## 工作中注意

用循环反复检查条件，避免虚假唤醒和条件被其他线程改变；notify 唤醒哪个等待线程不由业务代码直接指定。

## 手练

用单个共享条件协调生产者和消费者，检查等待、唤醒以及退出路径。

来源：[教程：使用wait和notify](https://liaoxuefeng.com/books/java/threading/synchronize/wait-notify/index.html)。
