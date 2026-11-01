# 使用Profiles

> **重要程度：高**｜**手练：需要**
> **重点**：环境配置和激活。

## 核心概念

Profile 用于按环境选择配置或 Bean，例如本地测试和生产环境。

## 工作中注意

确认激活顺序、默认值与外部配置优先级；不要把生产凭据写进配置仓库。

## 手练

准备两个测试 Profile，验证切换后实际连接地址和生效 Bean。

来源：[教程：使用Profiles](https://liaoxuefeng.com/books/java/springboot/profiles/index.html)。

## 关联知识与实践

创建一个最小 Boot 应用，新增健康检查和一个返回 JSON 的接口；分别用两个 Profile 提供不同配置，检查实际生效结果。编写接口测试并构建产物。

来源：[教程：Spring Boot开发](https://liaoxuefeng.com/books/java/springboot/index.html)；[Spring Boot 系统要求](https://docs.spring.io/spring-boot/system-requirements.html)。
