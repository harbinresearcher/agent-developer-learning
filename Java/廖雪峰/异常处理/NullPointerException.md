# NullPointerException

> **重要程度：高**｜**手练：需要**
> **重点**：null 来源、拆箱、定位和边界验证。

## 核心概念

对 `null` 引用调用实例方法、访问实例字段，或对值为 `null` 的包装类型拆箱等操作可能触发 `NullPointerException`。异常发生的位置不一定是产生 `null` 的位置，需要向前检查数据来源。

## 工作中注意

在系统边界验证必须字段；对确实可能缺失的数据明确表达和处理。不要对所有字段盲目判空后返回默认值，否则可能掩盖数据错误。日志不要输出敏感对象完整内容。

## 手练

构造一次 null 拆箱错误，结合异常栈定位；再在数据入口验证缺失值，比较错误信息。

来源：[教程：NullPointerException](https://liaoxuefeng.com/books/java/exception/null-pointer-exception/index.html)。
