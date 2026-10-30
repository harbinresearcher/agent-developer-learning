# StringJoiner

> **重要程度：中**｜**手练：建议**
> **重点**：分隔符、前后缀、空集合处理。

`StringJoiner` 适合把多个元素按统一分隔符组合；构造时可指定分隔符、前缀和后缀，逐项 `add` 后获取结果。与手写循环相比，它能减少“首尾多一个分隔符”的特殊处理。

如果数据已经是可迭代字符串，`String.join` 或 Stream 的 `Collectors.joining` 也可能更直接。空输入和 `null` 值按具体 API 契约处理，不要假定所有拼接方法行为相同。

手练：把三个标签拼成带方括号的逗号分隔文本，再测试零个和一个标签。

来源：[教程：StringJoiner](https://liaoxuefeng.com/books/java/oop/core/string-joiner/index.html)。
