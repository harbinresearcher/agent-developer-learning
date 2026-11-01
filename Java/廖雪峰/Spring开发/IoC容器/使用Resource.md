# 使用Resource

> **重要程度：中**｜**手练：需要**
> **重点**：classpath 资源、输入流、打包后的读取方式。

## Resource 是什么

`org.springframework.core.io.Resource` 是资源访问抽象。`classpath:/greeting.txt` 指类路径资源；`file:` 指文件系统资源。它与 `jakarta.annotation.Resource` 注解不是同一个东西：后者用于注入依赖，这篇讨论资源内容。

例子在 `@Bean` 参数上使用 `@Value` 注入资源句柄，再打开输入流、按 UTF-8 解码并关闭。Maven 会把 `src/main/resources` 下的资源复制到类路径，打包后可能位于 JAR 内。


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

将代码保存为 `src/main/java/SpringResourceDemo.java`：

```java
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.util.logging.Logger;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.Resource;
public class SpringResourceDemo {
    private static final Logger logger = Logger.getLogger(SpringResourceDemo.class.getName());
    static class ReaderService {
        private final Resource resource;
        ReaderService(Resource resource) { this.resource = resource; }
        String read() throws Exception {
            try (InputStream in = resource.getInputStream()) {
                return new String(in.readAllBytes(), StandardCharsets.UTF_8).trim();
            }
        }
    }
    @Configuration
    static class Config {
        @Bean ReaderService readerService(@Value("classpath:/greeting.txt") Resource resource) {
            return new ReaderService(resource);
        }
    }
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：读取类路径资源
         * 数据源：打包进类路径的 greeting.txt
         * 操作：1) 注入资源句柄 2) 使用 UTF-8 读取并关闭流
         * ======================================================================== */
        logger.info("开始读取资源");
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            // 1.1 使用容器注入资源
            ReaderService service = context.getBean(ReaderService.class);
            // 1.2 打开并读取资源内容
            System.out.println(service.read());
        }
        logger.info("资源读取完成");
    }
}
```

同时创建 `src/main/resources/greeting.txt`，内容为 `你好，资源`。

在项目根目录运行（PowerShell）：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" SpringResourceDemo
```

标准输出如下，日志可能单独显示：

```text
你好，资源
```

## 工作中怎么用

不要假设类路径资源一定能转为普通文件：在 JAR 内读取时使用 `getInputStream()` 更通用。资源句柄创建成功不代表资源一定存在；打开流可能失败，需要按业务处理缺失。示例使用 `readAllBytes()`，只适合大小可控的资源，大文件应分块读取并限制大小。

用户可控的资源路径需要限制来源，不能任意读取文件或访问 URL。外部可变配置通常不应写进不可变的应用 JAR。

## 动手与自查

1. 在 `src/main/resources/greeting.txt` 写入 `你好，资源`，先按运行命令验证，再从 JAR 类路径运行。
2. 把文件改名，观察读取阶段的异常。
3. 为什么本地开发时 `getFile()` 可用，并不能证明打包到 JAR 后也可用？

来源：[教程：使用Resource](https://liaoxuefeng.com/books/java/spring/ioc/resource/index.html)、[Spring Resource](https://docs.spring.io/spring-framework/reference/core/resources.html)。
