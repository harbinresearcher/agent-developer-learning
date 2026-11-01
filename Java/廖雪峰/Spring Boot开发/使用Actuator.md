# 使用Actuator

> **重要程度：高**｜**手练：需要**
> **重点**：健康端点、HTTP 状态、暴露范围和访问控制。

## Actuator 做什么

Actuator 提供健康状态、指标等运维端点，实际端点和暴露方式取决于依赖与配置。健康状态可供监控或部署平台使用，但检查什么依赖、是否公开细节需要明确设计。

例子添加 Actuator starter，并用一个只读取演示属性的 `HealthIndicator` 模拟健康或故障；没有连接数据库或其他外部服务。Boot 4 的健康接口位于 `org.springframework.boot.health.contributor`，旧教程的包名可能不同。

## 完整工程

使用 JDK 17、Maven 与 Spring Boot 4.1.2。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>health-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-actuator</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

将代码保存为 `src/main/java/learn/health/HealthDemo.java`：

```java
package learn.health;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.health.contributor.Health;
import org.springframework.boot.health.contributor.HealthIndicator;
import org.springframework.context.annotation.Bean;
@SpringBootApplication
public class HealthDemo {
    private static final Logger logger = LoggerFactory.getLogger(HealthDemo.class);
    @Bean HealthIndicator demoHealth(@Value("${demo.healthy:true}") boolean healthy) {
        return () -> healthy ? Health.up().build() : Health.down().build();
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动健康检查示例
         * 数据源：Actuator 配置和模拟健康状态
         * 操作：1) 创建应用容器 2) 启动本地 Web 服务
         * ======================================================================== */
        logger.info("开始启动健康检查示例");
        // 1.1 启动 Web 应用及 Actuator
        SpringApplication.run(HealthDemo.class, args);
        logger.info("健康检查示例启动完成");
    }
}
```

## 启动并请求

先确认 `java -version` 与 `mvn -v` 使用 JDK 17 或以上。PowerShell 在工程目录运行：

```powershell
mvn package
java -jar target/health-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/actuator/health
curl.exe -i http://127.0.0.1:8080/actuator/env
```

默认健康状态为 `UP`，HTTP 200；默认未通过 Web 暴露的 `env` 返回 404。响应中还可能有健康组等字段，具体随版本和配置变化。按 `Ctrl+C` 停止服务，再用下列命令模拟故障：

```powershell
java -jar target/health-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080 --demo.healthy=false
```

再次请求 `/actuator/health`，本例返回 `DOWN` 和 HTTP 503。完成后停止服务；端口被占用时应同时修改启动参数和请求地址。

## 工作中怎么用

- 本例绑定本机回环地址且没有配置认证，只用于练习；生产环境应根据实际网络和安全策略限制管理端点访问。
- 默认仅健康端点通过 HTTP 暴露。不要把暴露列表设为全部而不检查敏感信息和授权；健康细节的显示也应单独配置。
- 区分进程存活和服务就绪。是否把数据库、缓存等外部依赖纳入健康检查，要结合故障传播和部署平台行为决定。
- 默认状态映射中 `DOWN` 和 `OUT_OF_SERVICE` 对应 503；自定义状态映射后需核对默认映射是否仍符合预期。

## 动手与自查

1. 分别运行健康与故障两次，检查 JSON 中的 `status` 和 HTTP 状态码。
2. 在隔离环境查看 `/actuator/env` 默认是否暴露；解释为什么不能假定所有端点都能公开。
3. 本例的模拟指标没有连接外部依赖，为什么不能证明真实数据库健康？

来源：[教程：使用Actuator](https://liaoxuefeng.com/books/java/springboot/actuator/index.html)、[Spring Boot 端点](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html)。
