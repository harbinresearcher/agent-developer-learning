# XML、JSON 与数据交换

> **重要程度：高**｜**手练：需要**
> **重点**：JSON 序列化和反序列化、字段校验、XML 解析与安全边界。

JSON 是常见的数据交换格式；对象、数组、字符串、数字、布尔值和 `null` 是基本结构。应用通常用 Jackson 等库在 JSON 与 Java 对象之间转换；第三方库需要通过项目构建工具引入，并按项目版本阅读 API 文档。

外部数据不能因为能解析就视为可信：还要检查必填字段、范围、字符串长度和业务约束。日期格式、数值精度、字段更名与兼容性要明确约定；不要将对象内部的密码、令牌等字段直接序列化给客户端。

XML 可用 DOM 整体解析，也可用 SAX 事件式处理；文件较大时要考虑内存开销。解析不可信 XML 时需按所用解析器配置禁用不必要的外部实体和外部资源，避免 XXE。

## 手练

把一条包含姓名和年龄的数据转成 JSON 再读回，检查缺失字段和非法年龄；写一个测试，确保敏感字段没有出现在响应中。

来源：[教程：XML与JSON](https://liaoxuefeng.com/books/java/xml-json/index.html)；[OWASP XXE 防护](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)。
