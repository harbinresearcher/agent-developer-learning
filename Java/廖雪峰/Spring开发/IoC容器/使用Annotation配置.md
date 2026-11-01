# 使用Annotation配置

> **重要程度：高**｜**手练：需要**
> **重点**：组件扫描范围、注解角色与构造器注入。

## 常见注解各做什么

`@Component` 表示类可作为组件候选；`@Service`、`@Repository`、`@Controller` 表达常见分层职责，也是组件标记。`@Configuration` 声明配置类，`@Bean` 方法显式提供对象，适合第三方类等无法直接加组件注解的情况。

组件扫描从指定包查找候选。例子用 `basePackageClasses` 以一个类型所在的包作为扫描起点，避免硬编码包名；内部组件使用 `static`，可以独立实例化。项目中通常将组件写成各自的顶层类，包结构清晰可查。


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

将代码保存为 `src/main/java/learn/scan/AnnotationConfigDemo.java`：

```java
package learn.scan;
import java.util.logging.Logger;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
public class AnnotationConfigDemo {
    private static final Logger logger = Logger.getLogger(AnnotationConfigDemo.class.getName());
    @Repository
    static class UserRepository { String findName() { return "小明"; } }
    @Service
    static class UserService {
        private final UserRepository repository;
        UserService(UserRepository repository) { this.repository = repository; }
        String greet() { return "你好，" + repository.findName(); }
    }
    @Configuration
    @ComponentScan(basePackageClasses = AnnotationConfigDemo.class)
    static class Config { }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：扫描并调用组件
         * 数据源：learn.scan 包中的组件类
         * 操作：1) 启动配置类 2) 获取服务并调用
         * ======================================================================== */
        logger.info("开始启动容器");
        // 1.1 扫描并创建服务及其依赖
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            UserService service = context.getBean(UserService.class);
            // 1.2 验证构造器依赖已经注入
            System.out.println(service.greet());
        }
        logger.info("组件调用完成");
    }
}
```

在项目根目录运行（PowerShell）：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" learn.scan.AnnotationConfigDemo
```

标准输出如下，日志可能单独显示：

```text
你好，小明
```

## 注入何时发生

启动容器时加载配置并扫描组件，随后容器创建 `UserService`，发现其唯一构造器需要 `UserRepository`，再注入匹配的 Bean。只有一个构造器时一般不必加 `@Autowired`。类上写了注解不代表普通 `new` 出的对象自动由 Spring 管理。

## 工作中排错

检查启动配置是否加载、目标包是否在扫描范围内、类是否符合组件条件，再看依赖是否存在或存在多个候选。不要对默认包或过大的根包盲目扫描；多模块项目尤其要核对包边界。

## 动手与自查

1. 删除 `@Repository`，观察缺少依赖时启动失败；恢复后再运行。
2. 将仓储移到扫描范围之外，说明即使类上有注解为何仍找不到。
3. `@Bean` 和组件扫描各适合什么情况？

来源：[教程：使用Annotation配置](https://liaoxuefeng.com/books/java/spring/ioc/annotation-config/index.html)、[Spring 组件扫描](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html)。
