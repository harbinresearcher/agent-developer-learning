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

## 关联知识与实践

Stream 处理数据的一般链路是：数据源 → 零个或多个中间操作 → 终止操作。`map` 转换、`filter` 筛选，`reduce` 归约，`collect` 收集结果。中间操作通常是惰性的；没有终止操作，整个流水线通常不会开始消费。一个 Stream 消费后不能重复使用。

避免在流水线里依赖外部可变状态；并行 Stream 并不自动更快，也可能让共享状态和线程池使用变复杂。涉及文件等资源的 Stream 需按 API 契约关闭。

从订单列表中过滤有效订单、计算总金额、按状态分组；对比普通循环与 Stream，说明哪种更易读。尝试重复消费同一 Stream，观察结果。

来源：[教程：函数式编程](https://liaoxuefeng.com/books/java/functional/index.html)；[Java Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
