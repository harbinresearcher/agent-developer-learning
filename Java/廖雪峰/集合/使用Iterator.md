# 使用Iterator

> **重要程度：高**｜**手练：需要**
> **重点**：遍历状态与安全删除。

## 核心概念

Iterator 使用 `hasNext()` 检查后续元素、`next()` 获取下一个元素；增强 for 通常使用迭代协议遍历可迭代对象。

## 工作中注意

遍历期间通过集合自身结构修改方法删除可能触发快速失败；需要删除时使用该迭代器支持的 remove。快速失败不是线程安全保证。

## 手练

遍历列表并删除符合条件的元素，用迭代器方法完成后验证剩余结果。

来源：[教程：使用Iterator](https://liaoxuefeng.com/books/java/collection/iterator/index.html)。
