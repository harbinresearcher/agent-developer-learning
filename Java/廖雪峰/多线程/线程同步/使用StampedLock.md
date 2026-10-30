# 使用StampedLock

> **重要程度：低**｜**手练：按需**
> **重点**：乐观读、validate 与不可重入。

## 核心概念

StampedLock 支持读锁、写锁和乐观读；乐观读需要读取数据后验证 stamp，验证失败应按正确路径重读。

## 工作中注意

它不是可重入锁，使用和错误处理比普通锁复杂；只有在确有需求并经性能测试后使用。

## 手练

画出乐观读、validate 失败和回退读锁的步骤，解释为何验证不可省略。

来源：[教程：使用StampedLock](https://liaoxuefeng.com/books/java/threading/synchronize/stamped-lock/index.html)。
