# MVC高级开发

> **重要程度：高**｜**手练：需要**
> **重点**：请求映射、表单参数、ModelAndView、输入校验和视图转义。

## 从低层 Servlet 到 MVC

直接使用 Servlet 时需要自己读取请求、选择处理逻辑并生成响应。MVC 框架可将请求映射到控制器方法、绑定参数，再根据模型和视图渲染页面。Spring MVC 的 `DispatcherServlet` 负责协调这些步骤；业务代码不需要为每个页面继承 Servlet。

本例使用 POST 表单参数和 `ModelAndView`。输入合法返回页面，空白或过长返回 400。它只是页面演示，不创建真实账户。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>mvc-form</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-thymeleaf</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/form/FormDemo.java`：

```java
package learn.form;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;
@SpringBootApplication
@Controller
public class FormDemo {
    private static final Logger logger = LoggerFactory.getLogger(FormDemo.class);
    @PostMapping("/signup")
    public ModelAndView signup(@RequestParam(name = "name", defaultValue = "") String name) {
        /* ========================================================================
         * 步骤1：校验并构造页面模型
         * 数据源：POST 表单中的 name 字段
         * 操作：1) 校验输入 2) 返回视图和状态
         * ======================================================================== */
        logger.info("开始处理表单");
        try {
            // 1.1 去除首尾空白，限制长度
            String cleaned = name.strip();
            ModelAndView result = new ModelAndView("result");
            if (cleaned.isEmpty() || cleaned.length() > 40) {
                result.setStatus(HttpStatus.BAD_REQUEST);
                result.addObject("message", "名称无效");
            } else {
                // 1.2 示例仅返回页面，不创建账户
                result.addObject("message", "你好，" + cleaned);
            }
            return result;
        } finally {
            logger.info("表单处理结束");
        }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动表单示例
         * 数据源：MVC 配置和模板
         * 操作：1) 启动容器 2) 注册本地接口
         * ======================================================================== */
        logger.info("开始启动表单示例");
        // 1.1 启动本地 Web 服务
        SpringApplication.run(FormDemo.class, args);
        logger.info("表单示例启动完成");
    }
}
```

保存 `src/main/resources/templates/result.html`：

```html
<!doctype html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
<head><meta charset="UTF-8"><title>结果</title></head>
<body><p th:text="${message}">结果</p></body>
</html>
```

确认 `java -version` 与 `mvn -v` 使用 JDK 17 或以上。在工程目录运行：

```powershell
mvn package
java -jar target/mvc-form-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i -X POST --data-urlencode "name=Alice" http://127.0.0.1:8080/signup
curl.exe -i -X POST --data-urlencode "name= " http://127.0.0.1:8080/signup
```

第一条返回 200，页面包含 `你好，Alice`；第二条返回 400，页面包含 `名称无效`。完成后在服务窗口按 `Ctrl+C`；端口被占用时同步修改请求地址。

## 处理链和边界

`@PostMapping` 选择请求路径和方法；`@RequestParam` 读取表单字段；控制器校验输入并把 `message` 放进 `ModelAndView`；模板引擎渲染 `result.html`。`th:text` 对 HTML 文本内容转义，不能将其泛化为脚本、URL 等所有上下文的安全保证。

真实项目可使用专门的表单对象和 Bean Validation，但应限制可绑定字段，避免客户端覆盖不应修改的属性。POST 不自动防止重复提交或 CSRF；实际账户创建还需认证、授权、CSRF 防护、幂等和安全的持久化处理。

## 动手与自查

1. 测试空白、合法值和超过 40 个字符的值，核对状态码。
2. 传入包含 `<script>` 的名称，查看页面源码中的转义结果。
3. 为什么示例返回 200 的页面不能证明真实账户已创建？

来源：[教程：MVC高级开发](https://liaoxuefeng.com/books/java/web/mvc-adv/index.html)、[Spring MVC 控制器](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)。
