# MVC开发

> **重要程度：高**｜**手练：需要**
> **重点**：MVC 分工、请求流转、模型数据和模板渲染。

## MVC 分别负责什么

MVC 是划分职责的一种方式：Model 表示页面需要的数据与相关业务模型，View 负责呈现，Controller 接收请求、协调处理并选择响应。实际项目通常还有独立的 Service 与 Repository；MVC 不是要求所有业务都写在 Controller 中。

例子中 Controller 读取 `name` 参数，将其放入 Model，返回 `greeting` 视图名；Thymeleaf 使用模板生成 HTML。这里为了聚焦请求链，示例没有数据库或复杂业务服务。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2；先确认 `java -version` 和 `mvn -v` 均使用 JDK 17 或以上。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>mvc-view</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-thymeleaf</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/mvc/MvcViewDemo.java`：

```java
package learn.mvc;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
@SpringBootApplication
@Controller
public class MvcViewDemo {
    private static final Logger logger = LoggerFactory.getLogger(MvcViewDemo.class);
    @GetMapping("/hello")
    public String hello(@RequestParam(name = "name", defaultValue = "guest") String name, Model model) {
        /* ========================================================================
         * 步骤1：准备问候页面
         * 数据源：查询参数 name
         * 操作：1) 放入模型 2) 返回视图名
         * ======================================================================== */
        logger.info("开始准备问候页面");
        // 1.1 将数据交给视图，模板负责 HTML 转义
        model.addAttribute("name", name);
        logger.info("问候页面模型准备完成");
        // 1.2 返回模板名，而不是直接返回响应正文
        return "greeting";
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动 MVC 示例
         * 数据源：Boot 配置及控制器
         * 操作：1) 启动容器 2) 提供本地页面
         * ======================================================================== */
        logger.info("开始启动 MVC 示例");
        // 1.1 启动内嵌 Web 服务
        SpringApplication.run(MvcViewDemo.class, args);
        logger.info("MVC 示例启动完成");
    }
}
```

保存模板 `src/main/resources/templates/greeting.html`：

```html
<!doctype html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
<head><meta charset="UTF-8"><title>问候</title></head>
<body><p>你好，<span th:text="${name}">guest</span>！</p></body>
</html>
```

在工程目录运行：

```powershell
mvn package
java -jar target/mvc-view-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell 访问：

```powershell
curl.exe -i "http://127.0.0.1:8080/hello?name=Alice"
```

应得到 HTTP 200、HTML 内容，页面中包含 `你好，Alice！`。也可在浏览器打开同一地址。完成后在服务窗口按 `Ctrl+C`；端口被占用时同步修改启动参数与请求地址。

## 请求如何变成页面

访问 `/hello?name=Alice` 后，Spring MVC 找到控制器方法，将参数绑定为 `name`，再把 `name` 放进 Model。方法返回 `greeting` 后，视图解析与 Thymeleaf 模板处理生成 HTML，客户端收到渲染结果，而不是 Java Model 对象本身。

`th:text` 按 HTML 文本上下文转义内容。示例输入 `<script>` 时会显示为文本，不会把它当成 HTML 标签；这并不意味着所有 JavaScript、URL、CSS 等上下文都自动安全，仍需按输出位置处理。

## 工作中怎么用

控制器负责参数和响应边界；业务规则放在服务层，数据访问放在仓储层。对请求参数做必要的长度、类型和权限检查，不把外部输入直接拼进 SQL。服务端模板与前后端分离的 JSON API 是不同响应方式，项目可按需求选择。

## 动手与自查

1. 修改 `name` 参数并查看 HTML 源码，找出模板替换的位置。
2. 用含 `<`、`>` 的输入验证 HTML 文本转义。
3. 如果需要从数据库查询用户，为什么通常不应把 SQL 和业务规则都塞进控制器？

来源：[教程：MVC开发](https://liaoxuefeng.com/books/java/web/mvc/index.html)、[Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)。
