# 使用JDBC

> **重要程度：高**｜**手练：需要**
> **重点**：JdbcTemplate 与参数化。

## 核心概念

Spring JDBC 提供模板等方式减少连接、语句和结果集的样板代码。

## 工作中注意

值使用参数绑定；动态表名或排序字段需要白名单，异常处理仍要符合业务语义。

## 手练

通过 JdbcTemplate 查询测试记录，验证不存在记录和非法输入。

来源：[教程：使用JDBC](https://liaoxuefeng.com/books/java/spring/database/jdbc/index.html)。
