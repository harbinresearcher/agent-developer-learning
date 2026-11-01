# 使用Actuator

> **重要程度：高**｜**手练：建议**
> **重点**：健康检查与管理端点。

## 核心概念

Actuator 提供健康、指标等运维端点，具体可用内容由依赖和配置决定。

## 工作中注意

只暴露必要端点并控制访问；健康检查的内容和对外显示的信息应按环境配置。

## 手练

在测试环境启用健康端点，验证其访问权限和依赖异常时的状态。

来源：[教程：使用Actuator](https://liaoxuefeng.com/books/java/springboot/actuator/index.html)。

## 关联知识与实践

Actuator 可提供健康、指标等运维端点；是否暴露、暴露给谁以及健康状态是否包含外部依赖都需按项目配置。集成 OpenAPI、Redis 或消息系统时，核对版本兼容、连接池、超时和异常处理；消息消费还要设计重复投递与幂等。

来源：[教程：Spring Boot开发](https://liaoxuefeng.com/books/java/springboot/index.html)；[Spring Boot 系统要求](https://docs.spring.io/spring-boot/system-requirements.html)。
