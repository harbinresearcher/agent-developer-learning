# 集成MyBatis

> **重要程度：高**｜**手练：建议**
> **重点**：映射、参数与 SQL。

## 核心概念

MyBatis 将 SQL、参数和结果映射与 Java 方法结合，适合需要清楚控制 SQL 的场景。

## 工作中注意

值绑定与字符串替换有安全区别；动态标识符要白名单，复杂查询仍需查看索引和执行计划。

## 手练

编写带参数的查询映射，测试特殊字符输入和无结果情况。

来源：[教程：集成MyBatis](https://liaoxuefeng.com/books/java/spring/database/mybatis/index.html)。
