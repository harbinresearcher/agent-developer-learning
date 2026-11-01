# 定制Bean

> **重要程度：中**｜**手练：需要**
> **重点**：初始化、销毁回调和容器作用域边界。

## 生命周期是什么

容器管理的 Bean 从创建到可使用，再到容器关闭时清理，可能有初始化和销毁回调。例子使用 `@Bean(initMethod = "open", destroyMethod = "close")` 显式指定方法，启动时先输出初始化，业务使用后关闭容器，再输出销毁。

回调方法适合校验配置、初始化必要资源和释放连接等，但不能在启动阶段无界等待外部服务；失败路径也要处理已经分配的资源。实际应用应根据框架和部署方式设计超时、重试和关闭顺序。


## 完整例子

使用 JDK 17、Maven 和 Spring Framework 7.0.10。Spring 6 起要求 Java 17 或以上。新建目录，保存以下 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>learn.example</groupId><artifactId>spring-ioc-demo</artifactId><version>1.0</version>
    <properties><maven.compiler.release>17</maven.compiler.release><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding></properties>
    <dependencies>
        <dependency><groupId>org.springframework</groupId><artifactId>spring-context</artifactId><version>7.0.10</version></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

将代码保存为 `src/main/java/BeanLifecycleDemo.java`：

```java
import java.util.logging.Logger;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
public class BeanLifecycleDemo {
    private static final Logger logger = Logger.getLogger(BeanLifecycleDemo.class.getName());
    static class ExampleResource {
        void open() {
            logger.info("开始初始化资源");
            System.out.println("初始化");
            logger.info("资源初始化完成");
        }
        void use() { System.out.println("使用资源"); }
        void close() {
            logger.info("开始关闭资源");
            System.out.println("销毁");
            logger.info("资源关闭完成");
        }
    }
    @Configuration
    static class Config {
        @Bean(initMethod = "open", destroyMethod = "close")
        ExampleResource exampleResource() { return new ExampleResource(); }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：管理资源生命周期
         * 数据源：配置类定义的单例 Bean
         * 操作：1) 启动容器并使用 2) 关闭容器触发销毁
         * ======================================================================== */
        logger.info("开始创建容器");
        // 1.1 刷新容器时创建并初始化默认单例
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            context.getBean(ExampleResource.class).use();
        }
        // 1.2 离开作用域后容器关闭，触发销毁回调
        logger.info("容器关闭完成");
    }
}
```

在项目根目录运行（PowerShell）：

先运行 `java -version` 和 `mvn -v`，确认两者都使用 JDK 17 或以上；不一致时先调整 `JAVA_HOME` 和 `PATH`。

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" BeanLifecycleDemo
```

标准输出如下，日志可能单独显示：

```text
初始化
使用资源
销毁
```

## 作用域和责任

示例为默认 singleton，容器关闭时执行销毁回调。对 prototype，容器创建并初始化后，通常由调用方负责后续清理，不能假设容器关闭时会逐个销毁。外部手动 `new` 的对象也不会自动执行这些容器回调。

常见方式还包括 `@PostConstruct`、`@PreDestroy` 或框架接口；注解包及依赖随版本而变。Spring 6 及以上使用 Jakarta 命名空间的相关注解时，要确认所需 API 依赖可用。不要把不同方式的调用顺序当作未验证的业务前提。

## 动手与自查

1. 运行例子，记录初始化、使用、销毁的顺序。
2. 改为 prototype 并多次获取对象，验证容器关闭时的输出和清理责任。
3. 若 `open()` 在分配一部分资源后失败，应该由谁清理已经分配的部分？

来源：[教程：定制Bean](https://liaoxuefeng.com/books/java/spring/ioc/customize/index.html)、[Spring 生命周期回调](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html)。
