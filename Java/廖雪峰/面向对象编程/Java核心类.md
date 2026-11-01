# Java核心类

> **重要程度：高**｜**手练：需要**
> **重点**：字符串、包装类型、枚举与数值。

## 核心概念

Java 核心类提供字符串、包装类型、枚举、记录类及大数等常用能力。

## 工作中注意

关注不可变性、null、精度和 JDK 版本；具体行为应查当前项目使用的 API 文档。

## 手练

比较 String 内容、Integer 空值拆箱和 BigDecimal 的数值比较。

来源：[教程：Java核心类](https://liaoxuefeng.com/books/java/oop/core/index.html)。

## 关联知识与实践

- `String` 不可变；循环拼接较多字符串时可考虑 `StringBuilder`。有分隔符的拼接可用 `StringJoiner`。
- `Integer` 等包装类型可以与基本类型自动装箱、拆箱；对 `null` 拆箱会抛 `NullPointerException`。
- `enum` 适合表达有限状态；对外存储使用稳定的约定值，谨慎依赖序号 `ordinal()`。
- `BigInteger` 处理超出固定整型范围的整数；`BigDecimal` 处理需要十进制精度的数，金额通常从字符串构造，并明确舍入模式。
- `record` 可简化不可变数据载体定义，正式可用自 Java 16；是否使用取决于项目 JDK。

来源：[教程：Java核心类](https://liaoxuefeng.com/books/java/oop/core/index.html)；[Java records](https://docs.oracle.com/en/java/javase/17/language/records.html)。
