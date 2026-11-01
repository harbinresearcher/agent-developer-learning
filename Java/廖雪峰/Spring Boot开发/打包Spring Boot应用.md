# 打包Spring Boot应用

> **重要程度：高**｜**手练：需要**
> **重点**：Maven 打包、可执行 JAR、依赖布局、独立启动验证。

## 打包得到什么

Spring Boot Maven 插件可以把应用类和运行依赖重新打包为可执行 JAR。运行时用 `java -jar` 启动，不需要手工把每个依赖 JAR 加入命令行。本例使用 starter parent 与 Boot Maven 插件；父 POM 已为插件配置 `repackage` 执行，插件仍需在构建中声明。

普通 JAR 不一定是带依赖的 Boot 可执行 JAR；是否能用 `java -jar` 要看构建配置和清单。构建通过只说明当前构建步骤成功，本例没有单元测试，不代表生产部署或外部依赖已经验证。

## 创建可打包项目

使用 JDK 17、Maven、Spring Boot 4.1.2。先检查 `java -version` 与 `mvn -v` 均使用 JDK 17 或以上。在新建目录保存 `pom.xml`：

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

将代码保存为 `src/main/java/learn/boot/HelloApplication.java`：

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

## 构建和检查内容

在项目根目录运行：

```powershell
mvn package
jar tf target/hello-boot-1.0.jar | Select-String 'BOOT-INF/classes/learn/boot/HelloApplication.class|BOOT-INF/lib/'
```

可看到应用类位于 `BOOT-INF/classes/`，依赖位于 `BOOT-INF/lib/`。Boot 启动器读取嵌套依赖并调用应用的入口类；实际文件名与依赖版本以当前构建产物为准，不要手工修改生成的清单。

## 在独立目录启动

下面将生成的 JAR 复制到工作目录下的 `run-demo`，再从该目录启动。终端保持运行，在另一个终端请求接口：

```powershell
New-Item -ItemType Directory -Force run-demo
Copy-Item target/hello-boot-1.0.jar run-demo/app.jar
Push-Location run-demo
java -jar app.jar --server.address=127.0.0.1 --server.port=8080
```

```powershell
curl.exe -i http://127.0.0.1:8080/hello
```

应收到 HTTP 200，响应体为：

```json
{"message":"Hello, Spring Boot!"}
```

按 `Ctrl+C` 停止服务，然后执行 `Pop-Location` 返回工程目录。若 8080 被占用，启动参数和请求地址应改成同一个空闲端口。这里仅在本机回环地址验证，不涉及线上部署。

## 工作中怎么用

- 构建和运行的 JDK 大版本要兼容，确认目标环境的 JDK、系统权限与配置来源。
- 数据库、消息系统、外部服务等不会因为被打包就自动可用；应在目标环境验证连接、凭据和健康状态。
- 不要把生产密钥打进 JAR 或镜像；构建产物要有版本、来源和发布流程。
- `mvn package` 的测试阶段可能被跳过或没有测试；应看构建日志与项目的实际测试，而不是仅看成功字样。

## 动手与自查

1. 构建后列出 `BOOT-INF/classes` 和 `BOOT-INF/lib`，解释两者作用。
2. 在独立目录启动并请求接口，确认退出服务后端口释放。
3. 如果换到只安装旧版 Java 的机器，为什么同一个 JAR 可能无法运行？

来源：[教程：打包Spring Boot应用](https://liaoxuefeng.com/books/java/springboot/package/index.html)、[Spring Boot 可执行归档](https://docs.spring.io/spring-boot/maven-plugin/packaging.html)、[可执行 JAR 启动器](https://docs.spring.io/spring-boot/specification/executable-jar/launching.html)。
