# 使用DOM

> **重要程度：中**｜**手练：按需**
> **重点**：树形模型和内存。

## 核心概念

DOM 将文档解析为可导航的节点树，适合需要多次访问和修改结构的数据。

## 工作中注意

完整树会占用内存；解析不可信内容时需正确配置外部实体限制。

## 手练

解析小型 XML，读取指定元素；估计大文档为何会消耗更多内存。

来源：[教程：使用DOM](https://liaoxuefeng.com/books/java/xml-json/dom/index.html)。

## 关联知识与实践

XML 可用 DOM 整体解析，也可用 SAX 事件式处理；文件较大时要考虑内存开销。解析不可信 XML 时需按所用解析器配置禁用不必要的外部实体和外部资源，避免 XXE。

来源：[教程：XML与JSON](https://liaoxuefeng.com/books/java/xml-json/index.html)；[OWASP XXE 防护](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)。
