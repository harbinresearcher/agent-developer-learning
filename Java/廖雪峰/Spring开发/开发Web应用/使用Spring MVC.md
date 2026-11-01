# 使用Spring MVC

> **重要程度：高**｜**手练：需要**
> **重点**：控制器映射、参数绑定、Model、视图解析与安全输出。

## Spring MVC 的请求链

`DispatcherServlet` 是 Spring MVC 的前端控制器，负责协调请求映射、调用处理器和处理返回值。本例用 `@Controller` 声明处理器，`@GetMapping` 指定方法和路径，`@RequestParam` 读取查询参数，`Model` 携带页面数据。

控制器返回的 `greeting` 是视图名；Thymeleaf 根据配置找到 `templates/greeting.html` 并渲染。这里使用 Boot 简化容器和模板配置，理解请求链后仍需按实际项目版本检查 MVC 配置。

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

## 参数、视图和异常

不传 `name` 时示例使用 `guest`；传入 `Alice` 时 Model 保存 `Alice`，模板的 `th:text` 输出经过 HTML 文本转义的内容。`@Controller` 返回字符串通常表示视图名；若使用 `@RestController` 或 `@ResponseBody`，返回值通常作为响应体处理，不能混淆两者。

真实接口应校验参数长度和业务规则，统一处理参数错误与服务异常。请求对象到领域对象的绑定应控制允许字段，避免客户端覆盖不应修改的属性。模板转义不是所有上下文的通用安全方案，链接、脚本和 HTML 富文本有各自处理要求。

## 动手与自查

1. 不传参数、传 `Alice` 分别请求，比较页面中显示的名字。
2. 将模板名改错，观察视图解析失败；再恢复。
3. 把控制器改为 `@RestController` 后返回 `greeting`，预测响应会怎样变化。

来源：[教程：使用Spring MVC](https://liaoxuefeng.com/books/java/spring/web/mvc/index.html)、[Spring MVC 控制器](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)。
