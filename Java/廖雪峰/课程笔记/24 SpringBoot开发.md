# Spring Boot：启动、配置与运维

> **重要程度：高**｜**手练：需要**
> **重点**：自动配置、外部配置、Profiles、打包、健康检查和依赖版本。

Spring Boot 在 Spring 基础上提供自动配置、依赖管理和应用启动支持。自动配置按类路径、已有 Bean 和条件决定是否生效；它不是“无需配置”。排查时检查实际依赖、配置、条件评估和启动日志。

配置可来自配置文件、环境变量和其他外部来源；明确优先级与当前激活的 Profile。密码和令牌交给受控配置或密钥管理，不写入源码、镜像或日志。构建产物、启动参数和运行环境由项目约定决定；`mvn package` 成功不等于生产部署成功。

Actuator 可提供健康、指标等运维端点；是否暴露、暴露给谁以及健康状态是否包含外部依赖都需按项目配置。集成 OpenAPI、Redis 或消息系统时，核对版本兼容、连接池、超时和异常处理；消息消费还要设计重复投递与幂等。

不同 Spring Boot 大版本支持的 Java 和 Jakarta 版本不同。开始项目时查看该版本官方 system requirements，不把某个旧教程版本当作通用要求。

## 手练

创建一个最小 Boot 应用，新增健康检查和一个返回 JSON 的接口；分别用两个 Profile 提供不同配置，检查实际生效结果。编写接口测试并构建产物。

来源：[教程：Spring Boot开发](https://liaoxuefeng.com/books/java/springboot/index.html)；[Spring Boot 系统要求](https://docs.spring.io/spring-boot/system-requirements.html)。
