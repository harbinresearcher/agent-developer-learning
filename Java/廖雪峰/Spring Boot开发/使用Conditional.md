# 使用Conditional

> **重要程度：高**｜**手练：需要**
> **重点**：属性条件、默认行为、Bean 注册和失败排查。

## 条件装配解决什么问题

某项功能可能只在配置开启、依赖存在或环境匹配时需要。条件装配让容器决定是否注册 Bean，而不是在每次业务调用时都写分支。例子使用 Boot 的 `@ConditionalOnProperty`，只有 `feature.enabled=true` 才注册 `featureService`。


## 完整例子

使用 JDK 17、Maven 与 Spring Boot 4.1.2。先确认 `java -version` 和 `mvn -v` 均使用 JDK 17 或以上。在新目录保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>conditional-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/config/ConditionalDemo.java`：

```java
package learn.config;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
@SpringBootApplication
public class ConditionalDemo {
    private static final Logger logger = LoggerFactory.getLogger(ConditionalDemo.class);
    static class FeatureService { String message() { return "功能可用"; } }
    @Bean
    @ConditionalOnProperty(prefix = "feature", name = "enabled", havingValue = "true")
    FeatureService featureService() { return new FeatureService(); }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：检查条件装配
         * 数据源：feature.enabled 属性
         * 操作：1) 根据属性创建容器 2) 检查功能 Bean 是否存在
         * ======================================================================== */
        logger.info("开始检查功能开关");
        try (ConfigurableApplicationContext context = SpringApplication.run(ConditionalDemo.class, args)) {
            // 1.1 容器已按条件处理 Bean 定义
            boolean present = context.containsBean("featureService");
            // 1.2 输出实际装配结果
            System.out.println("功能Bean存在=" + present);
        }
        logger.info("功能开关检查结束");
    }
}
```

在项目根目录的 PowerShell 中运行：

```powershell
mvn package
java -jar target/conditional-demo-1.0.jar --feature.enabled=true
java -jar target/conditional-demo-1.0.jar --feature.enabled=false
java -jar target/conditional-demo-1.0.jar
```

下面列出每次运行的关键输出；其他启动日志省略：

```text
功能Bean存在=true
功能Bean存在=false
功能Bean存在=false
```

## 判断结果

传 `true` 时存在，传 `false` 或不传时不存在，因为本例的 `matchIfMissing` 沿用默认 `false`。不设置 `havingValue` 时默认判断规则并不等同于“只接受 true”：属性存在且不等于 `false` 也可能匹配。集合属性条件不能简单依赖此注解推断所有索引项。

## 工作中怎么用

调用方如果无条件依赖被关闭的 Bean，容器可能启动失败；要为关闭场景设计默认实现、可选依赖或同步关闭依赖方。多个条件同时成立可能出现多个候选，需要明确选择。

排错时核对最终属性值、条件注解所在的配置是否加载及实际 Bean；Boot 的条件评估报告可辅助诊断。条件装配发生在容器处理配置时，运行中修改配置值不意味着已创建 Bean 会自动重建。

## 动手与自查

1. 分别传 `true`、`false` 和不传，核对三个结果。
2. 改成 `matchIfMissing = true`，重新测试缺省行为。
3. 让另一个 Bean 强制依赖该服务，再关闭功能，观察为何启动失败。

来源：[教程：使用Conditional](https://liaoxuefeng.com/books/java/springboot/conditional/index.html)、[ConditionalOnProperty API](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/condition/ConditionalOnProperty.html)。
