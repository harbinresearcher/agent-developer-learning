# 使用Interceptor

> **重要程度：高**｜**手练：需要**
> **重点**：MVC 处理器拦截、路径匹配、放行与 Filter 边界。

## 拦截器在哪一层

Spring MVC 的 `HandlerInterceptor` 参与处理器调用过程。常用 `preHandle` 可在目标处理器执行前检查或准备数据；返回 `true` 继续，返回 `false` 则终止后续处理，拦截器需要确保响应符合约定。`postHandle` 和 `afterCompletion` 有各自的调用时机与异常边界。

示例通过 `WebMvcConfigurer` 注册拦截器，只匹配 `/items/**`，并添加固定演示响应头。`/ping` 不在匹配范围内。它只演示位置与路径，不实现认证。

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

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/items/7
curl.exe -i http://127.0.0.1:8080/ping
```

第一条返回 200 和 `{"id":7}`，响应头包含 `X-Demo-Interceptor: applied`；第二条返回 `pong`，不包含该演示头。完成后在服务窗口按 `Ctrl+C`；端口占用时同步修改启动参数与地址。

## 与 Filter 的关系

Filter 由 Servlet 容器的过滤链调用，覆盖范围取决于注册方式；Interceptor 由 Spring MVC 在选定处理器的流程中调用。静态资源、错误和异步分发是否经过某个组件，要看映射与配置，不应凭名称推断。

拦截器应保持职责单一，避免在共享实例字段中保存每次请求的数据。认证和授权通常优先使用项目的安全过滤链；仅靠 MVC 路径拦截可能遗漏其他入口。异步请求和异常时的回调顺序应结合框架版本测试。

## 动手与自查

1. 分别请求两个路径并比较响应头。
2. 在隔离示例中让 `preHandle` 设置 403 并返回 `false`，验证处理器是否执行。
3. 为什么安全控制不能只靠一个未经全面覆盖检查的 MVC 拦截器？

来源：[教程：使用Interceptor](https://liaoxuefeng.com/books/java/spring/web/interceptor/index.html)、[Spring MVC 拦截器](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-config/interceptors.html)。
