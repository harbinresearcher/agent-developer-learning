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

## 关联知识与实践

JDBC 提供连接数据库的标准接口，具体数据库需要对应驱动。常见流程是：从数据源取得连接 → 创建 `PreparedStatement` → 设置参数 → 执行 → 读取 `ResultSet` → 关闭资源。用参数占位符传入值，不拼接不可信输入；表名、列名等通常不能直接作为值参数，需要白名单控制。

来源：[教程：JDBC](https://liaoxuefeng.com/books/java/jdbc/index.html)；[JDBC 基础](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html)。
