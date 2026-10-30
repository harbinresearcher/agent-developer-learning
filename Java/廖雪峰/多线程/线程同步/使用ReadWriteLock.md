# 使用ReadWriteLock

> **重要程度：中**｜**手练：按需**
> **重点**：读多写少与互斥规则。

## 核心概念

读写锁允许多个读者并发持读锁，但写操作需要独占；适用于读多写少且测量显示有收益的共享状态。

## 工作中注意

读写锁并非天然更快；错误的锁升级或不一致的保护范围会导致问题，具体公平性依实现和配置。

## 手练

创建共享缓存，分别用读锁读取和写锁更新，验证读写期间的一致性。

来源：[教程：使用ReadWriteLock](https://liaoxuefeng.com/books/java/threading/synchronize/read-write-lock/index.html)。
