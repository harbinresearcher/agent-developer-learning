# 使用Profiles

> **重要程度：高**｜**手练：需要**
> **重点**：激活环境、选择配置文件与 Bean、优先级和缺失配置。

## Profile 解决什么问题

开发、测试、生产环境的服务地址和功能开关往往不同。Profile 可以按当前环境选择配置文件和 Bean。它只是配置选择机制，不自动提供安全隔离，也不代表名字叫 `prod` 的配置就是安全的。

例子中 `dev` 和 `prod` 各有一个属性文件和一个 `MessageService` 实现；只激活其中一个时，容器获取对应服务。本例不连接数据库或真实外部服务。

## 创建工程

使用 JDK 17、Maven 和 Spring Boot 4.1.2；先用 `java -version` 和 `mvn -v` 确认 Java 与 Maven 使用 JDK 17 或以上。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>profiles-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

代码保存为 `src/main/java/learn/profiles/ProfileDemo.java`：

```java
package learn.profiles;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Profile;
@SpringBootApplication
public class ProfileDemo {
    private static final Logger logger = LoggerFactory.getLogger(ProfileDemo.class);
    interface MessageService { String message(); }
    @Bean @Profile("dev") MessageService devService(@Value("${app.label}") String label) {
        return () -> "开发：" + label;
    }
    @Bean @Profile("prod") MessageService prodService(@Value("${app.label}") String label) {
        return () -> "生产：" + label;
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：读取当前环境配置
         * 数据源：启动参数与对应的 profile 配置文件
         * 操作：1) 启动容器 2) 获取匹配的服务并输出
         * ======================================================================== */
        logger.info("开始启动环境示例");
        // 1.1 启动容器并读取 profile 选择的 Bean
        try (ConfigurableApplicationContext context = SpringApplication.run(ProfileDemo.class, args)) {
            MessageService service = context.getBean(MessageService.class);
            // 1.2 输出当前配置，不连接外部系统
            System.out.println("环境结果=" + service.message());
        }
        logger.info("环境示例结束");
    }
}
```

再创建 `src/main/resources/application-dev.properties`：

```properties
app.label=dev-local
```

创建 `src/main/resources/application-prod.properties`：

```properties
app.label=prod-example
```

## 打包并切换环境

在项目根目录的 PowerShell 中运行：

```powershell
mvn package
java -jar target/profiles-demo-1.0.jar --spring.profiles.active=dev
java -jar target/profiles-demo-1.0.jar --spring.profiles.active=prod
```

日志之外，分别应看到：

```text
环境结果=开发：dev-local
环境结果=生产：prod-example
```

应用启动时读取激活的 profile，加载对应属性文件，再按 `@Profile` 注册匹配的 Bean；程序读取结果后关闭容器。示例没有启动 Web 服务器。

## 覆盖和边界

命令行也可以提供属性。例如下列命令中的 `--app.label` 覆盖本例 `application-dev.properties` 的同名值：

```powershell
java -jar target/profiles-demo-1.0.jar --spring.profiles.active=dev --app.label=from-cli
```

输出包含 `环境结果=开发：from-cli`。实际项目还可能从环境变量、外部文件等来源读取配置；优先级应核对所用 Boot 版本的官方文档。不要把生产密码放进示例文件或源码仓库，也不要把命令行当作安全的秘密存储。

如果没有激活 `dev` 或 `prod`，例子中没有匹配的 `MessageService`，获取 Bean 会失败；若同时激活两个，可能出现同类型候选歧义。真实项目要明确默认环境与冲突处理。Spring Boot 对 `spring.profiles.active` 可放置的配置文档有约束，不应随意写进 profile 专属文件。

## 动手与自查

1. 按两条命令切换环境，核对输出与配置文件。
2. 不传 profile，观察缺少 Bean 的异常；再思考怎样设计安全的默认行为。
3. 用命令行覆盖 `app.label`，解释为什么需要了解属性来源的优先级。

来源：[教程：使用Profiles](https://liaoxuefeng.com/books/java/springboot/profiles/index.html)、[Spring Boot Profiles](https://docs.spring.io/spring-boot/reference/features/profiles.html)、[外部化配置](https://docs.spring.io/spring-boot/reference/features/external-config.html)。
