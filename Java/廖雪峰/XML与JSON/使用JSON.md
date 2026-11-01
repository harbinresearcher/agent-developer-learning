# 使用JSON

> **重要程度：高**｜**手练：需要**
> **重点**：基本类型、对象与数组。

## 核心概念

JSON 支持对象、数组、字符串、数字、布尔值和 null，广泛用于服务间数据交换。

## 工作中注意

字段命名、数值精度、日期格式与未知字段行为应明确约定；JSON 解析成功并不代表请求合法。

## 手练

设计包含姓名和年龄的请求体，测试缺失字段和错误类型。

来源：[教程：使用JSON](https://liaoxuefeng.com/books/java/xml-json/json/index.html)。

## 关联知识与实践

JSON 是常见的数据交换格式；对象、数组、字符串、数字、布尔值和 `null` 是基本结构。应用通常用 Jackson 等库在 JSON 与 Java 对象之间转换；第三方库需要通过项目构建工具引入，并按项目版本阅读 API 文档。

来源：[教程：XML与JSON](https://liaoxuefeng.com/books/java/xml-json/index.html)；[OWASP XXE 防护](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)。
