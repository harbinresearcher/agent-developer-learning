# 集成JPA

> **重要程度：中**｜**手练：建议**
> **重点**：规范、实现与实体管理。

## 核心概念

JPA 定义 Java 持久化相关规范，Hibernate 等可作为实现；实体和关联由映射配置描述。

## 工作中注意

规范不保证所有数据库特性完全一致；注意事务、惰性加载和版本兼容。

## 手练

定义测试实体，保存后按主键查询，再验证事务回滚。

来源：[教程：集成JPA](https://liaoxuefeng.com/books/java/spring/database/jpa/index.html)。
