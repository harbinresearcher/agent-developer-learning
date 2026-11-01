# JDBC连接池

> **重要程度：高**｜**手练：需要**
> **重点**：连接复用、容量和归还。

## 核心概念

连接池复用数据库连接，减少重复创建连接的成本；应用从数据源借用连接并在完成后关闭以归还。

## 工作中注意

连接池容量受数据库承载能力限制；长期占用、未关闭或超时设置不当可能导致连接耗尽。

## 手练

使用测试连接池执行查询，确保正常和异常路径都归还连接。

来源：[教程：JDBC连接池](https://liaoxuefeng.com/books/java/jdbc/pool/index.html)。

## 关联知识与实践

批处理可以降低多次往返的开销，结果和失败处理依赖驱动。连接池复用连接，不等于无限连接；要设置容量、超时并及时归还，避免连接耗尽。

来源：[教程：JDBC](https://liaoxuefeng.com/books/java/jdbc/index.html)；[JDBC 基础](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html)。
