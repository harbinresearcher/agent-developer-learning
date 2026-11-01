# JDBC事务

> **重要程度：高**｜**手练：需要**
> **重点**：自动提交、commit 和 rollback。

## 核心概念

事务将多个数据库操作组合为一个工作单元；关闭自动提交后，成功时提交，失败时回滚。

## 工作中注意

事务不能自动覆盖外部 HTTP 或消息发送；连接归还前要按池或框架契约恢复状态并关闭资源。

## 手练

模拟转账第二步失败，检查两个账户修改是否一起回滚。

来源：[教程：JDBC事务](https://liaoxuefeng.com/books/java/jdbc/tx/index.html)。

## 关联知识与实践

单个业务操作涉及多次修改时，要明确事务边界。关闭自动提交后，成功时提交，失败时回滚，并在连接归还连接池前恢复或由框架管理连接状态。事务提供一致性保障，但不能替代唯一约束、并发控制和幂等设计。

在测试数据库里插入和查询数据，再模拟第二步失败，验证两步修改一起回滚。用恶意输入测试参数化查询，确认它被当作值处理。

来源：[教程：JDBC](https://liaoxuefeng.com/books/java/jdbc/index.html)；[JDBC 基础](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html)。
