# 开发Web应用

> **重要程度：高**｜**手练：需要**
> **重点**：请求路由、控制器与服务、响应状态、异常和验证。

## 一次请求经过什么

客户端发送 HTTP 请求，Web 服务器和 Servlet 容器接收请求；Spring MVC 根据方法和路径找到控制器。控制器绑定参数，调用服务处理业务，再将结果转换为响应。访问数据库时，服务通常调用数据访问层；本例使用内存仓储，便于单独理解链路。

视图渲染和 JSON API 都可以由 Spring MVC 处理。`@Controller` 常用于返回视图名；`@RestController` 的返回值通常作为响应体处理。不能仅凭类名判断状态码、认证或事务是否正确。

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
    <groupId>learn.example</groupId><artifactId>rest-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/rest/RestDemo.java`：

```java
package learn.rest;
import java.util.Optional;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
record Product(long id, String name) { }
interface ProductRepository { Optional<Product> findById(long id); }
@Service
class ProductService {
    private final ProductRepository repository;
    ProductService(ProductRepository repository) { this.repository = repository; }
    Optional<Product> find(long id) { return repository.findById(id); }
}
@RestController
class ProductController {
    private static final Logger logger = LoggerFactory.getLogger(ProductController.class);
    private final ProductService service;
    ProductController(ProductService service) { this.service = service; }
    @GetMapping("/products/{id}")
    public ResponseEntity<Product> get(@PathVariable("id") long id) {
        /* ========================================================================
         * 步骤1：查询商品
         * 数据源：路径参数和内存仓储
         * 操作：1) 查询商品 2) 根据结果设置响应
         * ======================================================================== */
        logger.info("开始查询商品");
        try {
            // 1.1 找到时返回 200 和 JSON；不存在时返回 404
            return service.find(id).map(ResponseEntity::ok).orElseGet(() -> ResponseEntity.notFound().build());
        } finally {
            logger.info("商品查询结束");
        }
    }
}
@SpringBootApplication
public class RestDemo {
    private static final Logger logger = LoggerFactory.getLogger(RestDemo.class);
    @Bean ProductRepository productRepository() {
        return id -> id == 1 ? Optional.of(new Product(1, "keyboard")) : Optional.empty();
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动接口示例
         * 数据源：内存仓储与控制器
         * 操作：1) 启动容器 2) 注册商品接口
         * ======================================================================== */
        logger.info("开始启动接口示例");
        // 1.1 启动内嵌 Web 服务
        SpringApplication.run(RestDemo.class, args);
        logger.info("接口示例启动完成");
    }
}
```

确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/rest-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/products/1
curl.exe -i http://127.0.0.1:8080/products/99
```

第一条返回 200 和包含 `id`、`name` 的 JSON；第二条返回 404。结束后在服务窗口按 `Ctrl+C`；端口占用时同步修改启动参数和请求地址。

## 看懂代码中的职责

`ProductController` 接收路径参数并决定 HTTP 状态；`ProductService` 委托查询；`ProductRepository` 定义数据访问接口，本例实现只是内存数据。对象通过构造器接收依赖。返回 404 的分支与成功分支都需要测试。

## 工作中怎么用

参数需要做类型、范围、长度和业务校验；认证确认身份，授权检查该身份是否可执行操作，两者都需要明确。统一异常处理和错误格式，日志记录必要的请求信息但不包含密码或令牌。

连接数据库、缓存或外部服务后，要设计超时、事务和失败处理；不能因为本地内存示例成功就认为生产依赖可用。接口测试应覆盖合法输入、缺失资源、非法输入和权限失败。

## 动手与自查

1. 比较存在与不存在商品的状态码和正文。
2. 请求非数字 ID，观察参数绑定失败；设计统一的错误响应。
3. 如果新增数据库仓储，哪些代码属于控制器、服务和数据访问层？

来源：[教程：开发Web应用](https://liaoxuefeng.com/books/java/spring/web/index.html)、[Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)。
