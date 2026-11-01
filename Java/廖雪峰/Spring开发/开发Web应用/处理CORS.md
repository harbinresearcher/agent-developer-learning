# 处理CORS

> **重要程度：高**｜**手练：需要**
> **重点**：浏览器同源限制、预检请求、允许来源与凭据边界。

## 跨域发生在哪里

浏览器的同源策略限制网页脚本跨来源读取响应。来源由协议、主机和端口组成；其中任一不同即为不同来源。CORS 是服务器通过响应头告诉浏览器哪些跨来源请求可被允许读取的一组机制。

例子只对 `/items/{id}` 允许来源 `https://client.example` 的 GET。浏览器可能先发送 OPTIONS 预检，询问方法和请求头是否允许；服务器响应相应 CORS 头后浏览器才继续实际请求。不是所有跨来源请求都会预检。

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
    <groupId>learn.example</groupId><artifactId>mvc-controls</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/mvccontrols/MvcControlsDemo.java`：

```java
package learn.mvccontrols;
import java.util.Map;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@SpringBootApplication
@RestController
public class MvcControlsDemo {
    private static final Logger logger = LoggerFactory.getLogger(MvcControlsDemo.class);
    @Bean WebMvcConfigurer mvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override public void addInterceptors(InterceptorRegistry registry) {
                registry.addInterceptor(new HandlerInterceptor() {
                    @Override public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
                        /* ========================================================================
                         * 步骤1：处理 MVC 拦截器
                         * 数据源：匹配 /items/** 的请求
                         * 操作：1) 添加演示响应头 2) 放行处理器
                         * ======================================================================== */
                        logger.info("开始执行 MVC 拦截器");
                        // 1.1 给响应添加固定头并允许继续
                        res.setHeader("X-Demo-Interceptor", "applied");
                        logger.info("MVC 拦截器执行结束");
                        return true;
                    }
                }).addPathPatterns("/items/**");
            }
        };
    }
    @CrossOrigin(origins = "https://client.example", methods = org.springframework.web.bind.annotation.RequestMethod.GET)
    @GetMapping("/items/{id}")
    public Map<String, Long> item(@PathVariable("id") long id) { return Map.of("id", id); }
    @GetMapping("/ping") public String ping() { return "pong"; }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动 MVC 控制示例
         * 数据源：拦截器配置与测试控制器
         * 操作：1) 启动容器 2) 注册本地接口
         * ======================================================================== */
        logger.info("开始启动 MVC 示例");
        // 1.1 启动嵌入式服务
        SpringApplication.run(MvcControlsDemo.class, args);
        logger.info("MVC 示例启动完成");
    }
}
```

## 本机验证

先确认 `java -version` 与 `mvn -v` 都使用 JDK 17 或以上。在工程目录运行：

```powershell
mvn package
java -jar target/mvc-controls-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell 模拟允许来源的预检：

```powershell
curl.exe -i -X OPTIONS -H "Origin: https://client.example" -H "Access-Control-Request-Method: GET" http://127.0.0.1:8080/items/7
```

应收到 200，响应头包含允许来源和 GET 方法。把 `Origin` 改为 `https://other.example`，本例返回 403。完成后在服务窗口按 `Ctrl+C`；端口占用时同步修改启动参数和地址。

curl 只是发送 HTTP 请求并显示头部，不执行浏览器的同源策略；预检结果不能代替真实浏览器的端到端验证。

## 工作中怎么用

按需要明确允许来源、方法和请求头，不要因为报错就开放所有来源。是否允许携带 Cookie 等凭据需要单独设计，并与具体允许来源、浏览器规则和服务端认证配置共同检查。

CORS 不等于认证或授权，也不是 CSRF 的完整防护：非浏览器客户端不受浏览器同源策略约束。网关、代理和应用都可能设置 CORS 头，重复或冲突配置需要在实际响应中排查。

## 动手与自查

1. 分别测试允许和不允许的 Origin，比较状态码与响应头。
2. 将预检方法改为 POST，观察结果，并解释例子只声明 GET 的含义。
3. 为什么服务端即使限制浏览器跨域，也仍需要身份认证和权限校验？

来源：[教程：处理CORS](https://liaoxuefeng.com/books/java/spring/web/cors/index.html)、[Spring MVC CORS](https://docs.spring.io/spring-framework/reference/web/webmvc-cors.html)。
