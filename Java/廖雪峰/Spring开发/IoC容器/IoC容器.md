# IoC容器

> **重要程度：高**｜**手练：需要**
> **重点**：Bean、容器与依赖注入。

## 核心概念

Spring IoC 容器根据配置创建、组装和管理 Bean，并处理依赖关系。

## 工作中注意

容器管理对象不意味着线程安全；确认 Bean 的作用域和可变状态。

## 手练

创建包含服务和仓储的测试容器，确认依赖由容器注入。

来源：[教程：IoC容器](https://liaoxuefeng.com/books/java/spring/ioc/index.html)。

## 关联知识与实践

Spring 容器负责创建和管理 Bean，并按配置注入依赖。构造器注入让必需依赖在创建时明确；自动装配、条件装配、配置属性和不同 scope 都会影响最终对象。排错时先看容器实际创建了哪些 Bean，以及当前激活的配置。

来源：[教程：Spring开发](https://liaoxuefeng.com/books/java/spring/index.html)；[Spring 事务代理](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)。
