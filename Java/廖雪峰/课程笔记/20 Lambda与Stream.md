# Lambda 与 Stream

> **重要程度：高**｜**手练：需要**
> **重点**：函数式接口、方法引用、Stream 中间与终止操作、惰性和副作用。

函数式接口只有一个抽象方法，lambda 可作为该接口的实现；方法引用是适用时更简洁的写法。捕获的局部变量必须是 final 或事实上的 final，避免把它与对象内部状态是否可变混为一谈。

Stream 处理数据的一般链路是：数据源 → 零个或多个中间操作 → 终止操作。`map` 转换、`filter` 筛选，`reduce` 归约，`collect` 收集结果。中间操作通常是惰性的；没有终止操作，整个流水线通常不会开始消费。一个 Stream 消费后不能重复使用。

避免在流水线里依赖外部可变状态；并行 Stream 并不自动更快，也可能让共享状态和线程池使用变复杂。涉及文件等资源的 Stream 需按 API 契约关闭。

## 手练

从订单列表中过滤有效订单、计算总金额、按状态分组；对比普通循环与 Stream，说明哪种更易读。尝试重复消费同一 Stream，观察结果。

来源：[教程：函数式编程](https://liaoxuefeng.com/books/java/functional/index.html)；[Java Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
