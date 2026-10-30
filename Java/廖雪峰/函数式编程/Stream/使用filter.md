# 使用filter

> **重要程度：高**｜**手练：需要**
> **重点**：谓词筛选和惰性。

## 核心概念

filter 根据谓词保留符合条件的元素，返回一个新的 Stream 流水线阶段。

## 工作中注意

没有终止操作时，过滤逻辑通常不会运行；避免在谓词中承担依赖执行次数的副作用。

## 手练

过滤出及格成绩并收集，验证空输入和没有符合条件的输入。

来源：[教程：使用filter](https://liaoxuefeng.com/books/java/functional/stream/filter/index.html)。
