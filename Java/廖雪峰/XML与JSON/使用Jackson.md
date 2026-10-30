# 使用Jackson

> **重要程度：高**｜**手练：建议**
> **重点**：对象映射、依赖与字段控制。

## 核心概念

Jackson 可按配置把 JSON 等数据与 Java 对象互相转换；需要通过构建工具引入与项目兼容的依赖。

## 工作中注意

反序列化不等于业务校验；限制暴露敏感字段，并对不可信输入避免不安全的多态类型配置。

## 手练

将测试对象转换为 JSON 再读回，验证缺失字段、非法输入和敏感字段处理。

来源：[教程：使用Jackson](https://liaoxuefeng.com/books/java/xml-json/jackson/index.html)。
