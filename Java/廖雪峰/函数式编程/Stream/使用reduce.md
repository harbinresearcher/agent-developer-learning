# 使用reduce

> **重要程度：中**｜**手练：建议**
> **重点**：初始值、累积和结合律。

## 核心概念

reduce 将流元素归约成单个结果，可使用初始值和累积函数；无初始值时通常返回 Optional。

## 工作中注意

并行归约要求操作满足相关结合性和无干扰约束；简单求和优先使用清晰的专门 API。

## 手练

求一组数的总和并测试空流，解释初始值如何影响结果。

来源：[教程：使用reduce](https://liaoxuefeng.com/books/java/functional/stream/reduce/index.html)。
