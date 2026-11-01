# 使用REST

> **重要程度：高**｜**手练：需要**
> **重点**：资源路径、HTTP 方法、JSON、状态码、分层和输入边界。

## 从一个接口理解

HTTP API 通常用路径表示资源，用方法表达操作，用状态码和响应体表达结果。本例提供 `GET /products/{id}`：编号为 1 时返回商品 JSON，不存在时返回 404。

`ProductController` 负责 HTTP 处理，`ProductService` 调用仓储接口，仓储返回内存中的测试数据。这个示例没有连接真实数据库，也没有认证；拆层是为了看清依赖方向，不意味着每个小程序都必须有同样多的类。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2；先确认 `java -version` 与 `mvn -v` 均使用 JDK 17 或以上。新建目录，保存 `pom.xml`：

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

在工程目录运行：

```powershell
mvn package
java -jar target/rest-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开 PowerShell：

```powershell
curl.exe -i http://127.0.0.1:8080/products/1
curl.exe -i http://127.0.0.1:8080/products/99
```

第一条返回 HTTP 200 和 JSON，内容包含 `id: 1`、`name: keyboard`；第二条返回 404。结束后在服务窗口按 `Ctrl+C`；端口被占用时同步修改启动参数和请求地址。

## 看懂参数和返回值

`@RestController` 将方法返回值作为响应体处理；`@PathVariable("id")` 将路径片段绑定为 long；`ResponseEntity` 明确表达状态码和正文。不存在的商品返回 404，而不是把不存在包装成一个看似成功的 200。路径不能转换为数字时由框架处理为客户端错误，生产接口可统一错误格式。

`GET`、`POST`、`PUT`、`DELETE` 各有语义；状态码、分页、错误结构和版本演进应由接口契约明确。重复付款或创建订单不能仅凭 HTTP 方法名保证安全，仍需服务端幂等设计。

## 工作中怎么用

对输入做类型、范围、长度和权限校验；不要把数据库实体和敏感字段无条件直接返回。区分认证与授权，记录必要的请求标识，避免在 URL 或日志中放令牌。真实仓储的连接、事务、索引和异常行为需要在目标数据库验证。

## 动手与自查

1. 分别请求存在和不存在的商品，核对状态码与响应类型。
2. 请求 `/products/abc`，观察数字转换失败的状态。
3. 新增创建商品接口前，先写出请求体、成功状态码、重复请求和错误响应的约定。

来源：[教程：使用REST](https://liaoxuefeng.com/books/java/spring/web/rest/index.html)、[Spring REST 控制器](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann.html)。
