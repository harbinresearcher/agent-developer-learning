# IoC原理

> **重要程度：高**｜**手练：需要**
> **重点**：构造器注入、依赖方向、测试替身与控制反转。

## 先看问题

如果 `UserService` 内部写死 `new MysqlUserRepository()`，服务负责业务，还负责决定依赖如何创建；单元测试时也容易被迫连接数据库。把依赖作为构造参数传入后，服务只依赖 `UserRepository` 接口，外部决定注入真实仓储还是测试替身。

控制反转（IoC）强调把对象创建或流程控制交给外部；依赖注入（DI）是其中一种具体做法。本例故意不使用 Spring：DI 先是一种设计方式，容器只是自动管理和组装对象的工具。


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

代码保存为 `src/main/java/IocPrincipleDemo.java`：

```java
import java.util.logging.Logger;
public class IocPrincipleDemo {
    private static final Logger logger = Logger.getLogger(IocPrincipleDemo.class.getName());
    interface UserRepository { String findName(int id); }
    static final class UserService {
        private final UserRepository repository;
        UserService(UserRepository repository) { this.repository = repository; }
        String greet(int id) { return "你好，" + repository.findName(id); }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：在应用边界组装依赖
         * 数据源：内存中的模拟用户仓储
         * 操作：1) 提供接口实现 2) 注入服务并调用
         * ======================================================================== */
        logger.info("开始组装服务");
        // 1.1 手工提供测试替身，服务不自己创建仓储
        UserRepository fake = id -> "小明";
        UserService service = new UserService(fake);
        // 1.2 调用服务并检查结果
        System.out.println(service.greet(7));
        logger.info("服务调用完成");
    }
}
```

在项目根目录运行（PowerShell）：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" IocPrincipleDemo
```

标准输出如下，框架和示例日志可能单独显示：

```text
你好，小明
```

## 怎么读例子

`UserService` 声明自己需要的仓储；主方法创建一个返回固定名称的替身，再把它传给构造器。调用 `greet` 时，服务只调用接口。测试替身可以模拟各种返回值和异常，不依赖真实数据库。

## 工作中的选择

优先用构造器接收必需依赖，并用 `final` 表明引用不应再被替换；可选依赖可以结合项目约定使用其他注入方式。构造器参数过多可能说明职责过多。不要把容器传给业务类，再到处调用 `getBean` 查找依赖，否则依赖会变隐蔽。

依赖接口有助于替换实现，但不应为了每一个不会变化的类机械地创建接口。依赖注入也不自动保证线程安全或事务生效。

## 动手与自查

1. 将替身改成返回另一个名字，验证不修改 `UserService` 就能改变结果。
2. 再写一个会抛异常的仓储，观察服务行为，设计需要的异常边界。
3. 对比服务内部 `new` 仓储和构造器接收仓储，谁负责选择实现？

来源：[教程：IoC原理](https://liaoxuefeng.com/books/java/spring/ioc/basic/index.html)、[Spring 依赖注入](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)。
