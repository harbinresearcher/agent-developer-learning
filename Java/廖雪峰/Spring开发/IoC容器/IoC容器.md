# IoC容器

> **重要程度：高**｜**手练：需要**
> **重点**：Bean 定义、容器管理、作用域与容器关闭。

## 容器与 Bean

Spring IoC 容器根据配置创建、组装和管理对象。由容器管理的对象称为 Bean；Bean 定义描述如何创建对象、名称、作用域和依赖。`ApplicationContext` 在基础 Bean 工厂能力上提供事件、资源访问等功能。

例子用 `@Configuration` 和 `@Bean` 声明一个对象，再启动 `AnnotationConfigApplicationContext`。`try-with-resources` 在结束时关闭容器，触发符合条件的容器关闭回调。


## 完整例子

使用 JDK 17、Maven 和 Spring Framework 7.0.10。Spring 6 及以上要求 Java 17 或更高版本；当前项目如使用其他版本，应遵循其依赖管理与兼容要求。新建空目录，将下列内容保存为 `pom.xml`：

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

代码保存为 `src/main/java/IocContainerDemo.java`：

```java
import java.util.logging.Logger;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
public class IocContainerDemo {
    private static final Logger logger = Logger.getLogger(IocContainerDemo.class.getName());
    static class GreetingService { String greet() { return "你好，Spring"; } }
    @Configuration
    static class Config {
        @Bean GreetingService greetingService() { return new GreetingService(); }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动容器并获取 Bean
         * 数据源：配置类中的 Bean 定义
         * 操作：1) 创建容器 2) 比较两次获取的对象
         * ======================================================================== */
        logger.info("开始创建容器");
        // 1.1 容器读取配置并创建默认作用域的 Bean
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            GreetingService first = context.getBean(GreetingService.class);
            GreetingService second = context.getBean(GreetingService.class);
            // 1.2 检查业务返回值与对象引用
            System.out.println(first.greet());
            System.out.println("同一 Bean: " + (first == second));
        }
        logger.info("容器已关闭");
    }
}
```

在项目根目录运行（PowerShell）：

先运行 `java -version` 和 `mvn -v`，确认两者都使用 JDK 17 或以上；不一致时先调整 `JAVA_HOME` 和 `PATH`。

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" IocContainerDemo
```

标准输出如下，框架和示例日志可能单独显示：

```text
你好，Spring
同一 Bean: true
```

## 理解作用域

默认 `singleton` 表示同一容器中同一 Bean 定义通常共享一个实例，所以本例两次获取相同引用；不同容器、不同 Bean 定义或不同进程不共享这一保证。`prototype` 每次向容器请求该 Bean 时通常创建新实例，销毁生命周期也与 singleton 不同。

共享对象不等于并发安全；无状态服务通常更容易安全共享，可变字段必须考虑并发。`new GreetingService()` 在容器外创建的对象不会自动成为容器管理的 Bean。

## 工作中排错

找不到 Bean 时先核对配置是否加载、组件是否位于扫描范围、条件是否成立；多个同类型 Bean 则检查限定符或主候选。启动成功也不能证明所有延迟创建的 Bean 都能正常创建。业务代码通常由容器注入依赖，而不是到处手写 `getBean`。

## 动手与自查

1. 添加第二个同类型 Bean，再按类型获取，观察歧义并用名称获取。
2. 将 Bean 标记 `@Scope("prototype")`，比较两次引用。
3. 为什么容器中的 singleton 不代表整个集群只有一个实例？

来源：[教程：IoC容器](https://liaoxuefeng.com/books/java/spring/ioc/index.html)、[Spring Bean 作用域](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)。
