# 第一个Spring Boot应用

> **重要程度：高**｜**手练：需要**
> **重点**：依赖、启动与接口。

## 核心概念

Spring Boot 提供应用启动和自动配置机制；一个最小 Web 应用通常包含构建依赖、入口类和请求处理器。

## 工作中注意

先确认所选 Boot 版本要求的 JDK 与依赖；启动成功还需通过实际请求验证。

## 手练

创建最小应用，运行一个返回 JSON 的接口并检查状态码。

来源：[教程：第一个Spring Boot应用](https://liaoxuefeng.com/books/java/springboot/first-webapp/index.html)。

## 关联知识与实践

Spring Boot 在 Spring 基础上提供自动配置、依赖管理和应用启动支持。自动配置按类路径、已有 Bean 和条件决定是否生效；它不是“无需配置”。排查时检查实际依赖、配置、条件评估和启动日志。

不同 Spring Boot 大版本支持的 Java 和 Jakarta 版本不同。开始项目时查看该版本官方 system requirements，不把某个旧教程版本当作通用要求。

来源：[教程：Spring Boot开发](https://liaoxuefeng.com/books/java/springboot/index.html)；[Spring Boot 系统要求](https://docs.spring.io/spring-boot/system-requirements.html)。
