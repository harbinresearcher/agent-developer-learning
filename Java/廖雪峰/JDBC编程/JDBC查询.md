# JDBC查询

> **重要程度：高**｜**手练：需要**
> **重点**：PreparedStatement 与 ResultSet。

## 核心概念

参数化查询先准备 SQL，再绑定参数，执行后通过 ResultSet 读取记录。读取的列类型要与数据库和代码的期望一致。

## 工作中注意

用户输入作为参数而不是拼接 SQL；查询结果可能为空或很大，应有明确处理和必要的分页。

## 手练

按用户编号查询测试数据，分别验证存在、不存在及特殊字符输入。

来源：[教程：JDBC查询](https://liaoxuefeng.com/books/java/jdbc/query/index.html)。
