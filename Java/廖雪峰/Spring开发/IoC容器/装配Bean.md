# 装配Bean

> **重要程度：高**｜**手练：需要**
> **重点**：构造器注入、同类型候选、@Qualifier 和循环依赖。

## 装配是什么意思

一个对象依赖另一个对象时，装配就是把依赖交给它。Spring 根据 Bean 定义、类型和限定条件找到候选，再调用构造器、工厂方法或属性注入。例子中 `OrderService` 通过构造器接收 `Notifier`，配置方法指定 `@Qualifier("email")`，容器注入名为 `email` 的候选。


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

代码保存为 `src/main/java/BeanWiringDemo.java`：

```java
import java.util.logging.Logger;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
public class BeanWiringDemo {
    private static final Logger logger = Logger.getLogger(BeanWiringDemo.class.getName());
    interface Notifier { String send(); }
    static class OrderService {
        private final Notifier notifier;
        OrderService(Notifier notifier) { this.notifier = notifier; }
        String place() { return notifier.send(); }
    }
    @Configuration
    static class Config {
        @Bean Notifier email() { return () -> "邮件通知"; }
        @Bean Notifier sms() { return () -> "短信通知"; }
        @Bean OrderService orderService(@Qualifier("email") Notifier notifier) {
            return new OrderService(notifier);
        }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：创建订单服务
         * 数据源：容器中的两个通知器 Bean
         * 操作：1) 按限定符选择依赖 2) 调用服务
         * ======================================================================== */
        logger.info("开始装配服务");
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            // 1.1 容器创建服务并注入指定通知器
            OrderService service = context.getBean(OrderService.class);
            // 1.2 验证使用的是邮件通知器
            System.out.println(service.place());
        }
        logger.info("服务调用完成");
    }
}
```

在项目根目录运行（PowerShell）：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" BeanWiringDemo
```

标准输出如下，框架和示例日志可能单独显示：

```text
邮件通知
```

## 候选冲突与循环依赖

当多个 Bean 都实现相同接口且缺少明确选择时，容器可能无法确定候选并报告歧义。可按项目约定使用 `@Qualifier` 或设置一个 `@Primary`；不要依赖不清晰的偶然选择。依赖被声明为集合时，语义是获取多个候选，与要求单个候选不同。

若 A 的构造器需要 B，B 的构造器又需要 A，容器无法完成这条构造链，通常会报循环依赖。优先重新划分职责，而不是盲目加入延迟加载掩盖设计问题。

## 工作中怎么用

必需依赖使用构造器注入，引用可声明 `final`；只有一个构造器的 Spring Bean 通常无需额外标注 `@Autowired`。框架注入只作用于容器管理的对象；手动 `new` 的对象不会因为字段上有注解就自动获得依赖。

## 动手与自查

1. 去掉限定符，运行并观察同类型候选导致的异常；再改成 `sms` 验证输出。
2. 将一个通知器设为 `@Primary`，去掉限定符，确认选择结果。
3. 为什么在服务类内部调用 `new` 得到的依赖不受容器管理？

来源：[教程：装配Bean](https://liaoxuefeng.com/books/java/spring/ioc/injection/index.html)、[Spring Qualifier](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)。
