# 使用Stream

> **重要程度：高**｜**手练：需要**
> **重点**：数据源、中间操作与终止操作。

## 核心概念

Stream 将数据处理组织为数据源、中间操作和终止操作；中间操作通常惰性执行。

## 工作中注意

Stream 消费后不能重复使用；涉及文件等资源时按 API 契约关闭，避免依赖外部可变状态。

## 手练

用 Stream 过滤、转换并收集测试列表，再尝试重复消费同一个 Stream。

来源：[教程：使用Stream](https://liaoxuefeng.com/books/java/functional/stream/index.html)。
