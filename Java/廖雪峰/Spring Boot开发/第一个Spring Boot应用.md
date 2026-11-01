# 第一个Spring Boot应用

> **重要程度：高**｜**手练：需要**
> **重点**：依赖管理、启动类、HTTP 接口、打包与本地验证。

## Spring Boot 是什么

Spring Framework 提供容器、MVC、事务等基础能力；Spring Boot 在其上提供依赖管理、自动配置和应用启动方式。自动配置根据类路径、配置和已有 Bean 等条件决定是否生效，不等于完全不需要配置。

下面创建一个只有一个 GET 接口的应用：启动内嵌 Web 服务器，访问 `/hello` 返回 JSON。它没有数据库、登录和权限控制，仅供本机学习。

## 准备工程

本例使用 JDK 17、Maven 3.6.3 或以上和 Spring Boot 4.1.2。Boot 4 使用 Java 17 或以上；旧项目的版本与依赖要看项目自身，不要直接混用大版本。先运行 `java -version`、`mvn -v`，确认 Java 与 Maven 均使用 JDK 17 或以上。

新建空目录，保存以下 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>hello-boot</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

`spring-boot-starter-parent` 管理插件和依赖版本；Boot 4 的 `spring-boot-starter-webmvc` 提供本例需要的 Web MVC 及嵌入式服务器相关依赖。不同 Boot 版本的 starter 命名可能不同，应查对应版本文档。

## 写启动类和接口

保存为 `src/main/java/learn/boot/HelloApplication.java`：

```java
package learn.boot;

import java.util.Map;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class HelloApplication {
    private static final Logger logger = LoggerFactory.getLogger(HelloApplication.class);

    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动应用
         * 数据源：启动参数与 Spring Boot 自动配置
         * 操作：1) 创建应用容器 2) 启动内嵌 Web 服务
         * ======================================================================== */
        logger.info("开始启动应用");
        // 1.1 根据类路径和配置启动容器
        SpringApplication.run(HelloApplication.class, args);
        logger.info("应用启动完成");
    }

    @GetMapping("/hello")
    public Map<String, String> hello() {
        /* ========================================================================
         * 步骤1：处理问候请求
         * 数据源：GET /hello
         * 操作：1) 构造响应 2) 返回 JSON 数据
         * ======================================================================== */
        logger.info("开始处理问候请求");
        // 1.1 构造固定响应，不访问外部服务
        Map<String, String> result = Map.of("message", "Hello, Spring Boot!");
        logger.info("问候请求处理完成");
        return result;
    }
}
```

`@SpringBootApplication` 组合应用配置、自动配置和组件扫描；把启动类放在应用包的上层，便于扫描其子包。`@RestController` 让方法返回值作为响应内容，`@GetMapping("/hello")` 映射 GET 路径。示例返回的 `Map` 会转换成 JSON，不是把 Java 对象的 `toString()` 当响应。

## 打包并验证

在项目根目录的 PowerShell 中运行：

```powershell
mvn package
java -jar target/hello-boot-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

保持窗口运行，另开一个 PowerShell 窗口：

```powershell
curl.exe -i http://127.0.0.1:8080/hello
```

应看到 HTTP 200 和类似下面的响应体：

```json
{"message":"Hello, Spring Boot!"}
```

完成后回到服务窗口按 `Ctrl+C` 停止。若 8080 被占用，可把启动参数和请求地址改成相同的其他空闲端口。示例明确绑定回环地址，便于本机练习；它没有实现生产环境所需的认证和部署配置。

## 从请求到响应

浏览器或客户端发送 GET 请求，内嵌服务器接收请求，Spring MVC 根据路径找到 `hello()`，调用方法后将返回值写成 JSON。启动日志出现不代表接口一定正确，必须实际请求并检查状态码和响应内容。

## 常见问题和自查

- `UnsupportedClassVersionError`：先检查运行时 `java -version`，它可能与 Maven 编译所用 JDK 不同。
- 404：核对请求方法、路径、应用端口、包扫描范围以及是否确实启动了当前应用。
- 启动失败提示端口占用：换端口或处理占用端口的进程，不要随意关闭不明程序。
- 修改返回消息并重新打包验证；再把路径改为 `/greeting`，确认旧路径不再匹配。
- 思考：为什么 `mvn package` 成功但不发请求，不能证明 HTTP 接口正常？

来源：[教程：第一个Spring Boot应用](https://liaoxuefeng.com/books/java/springboot/first-webapp/index.html)、[Spring Boot 首个应用](https://docs.spring.io/spring-boot/tutorial/first-application/)、[系统要求](https://docs.spring.io/spring-boot/system-requirements.html)。
