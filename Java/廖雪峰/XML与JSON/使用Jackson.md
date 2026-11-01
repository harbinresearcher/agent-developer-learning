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

## 关联知识与实践

外部数据不能因为能解析就视为可信：还要检查必填字段、范围、字符串长度和业务约束。日期格式、数值精度、字段更名与兼容性要明确约定；不要将对象内部的密码、令牌等字段直接序列化给客户端。

把一条包含姓名和年龄的数据转成 JSON 再读回，检查缺失字段和非法年龄；写一个测试，确保敏感字段没有出现在响应中。

来源：[教程：XML与JSON](https://liaoxuefeng.com/books/java/xml-json/index.html)；[OWASP XXE 防护](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)。
