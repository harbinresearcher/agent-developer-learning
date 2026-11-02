# 集成Open API

> **重要程度：高**｜**手练：需要**
> **重点**：OpenAPI 描述、文档生成、接口验证与访问控制。

## OpenAPI 做什么

OpenAPI 是描述 HTTP API 的规范，可以表达路径、方法、参数、响应和数据结构。springdoc 根据 Spring 应用配置与接口生成描述；Swagger UI 等工具可展示这些描述。文档帮助沟通，但不自动证明接口实现、鉴权或错误处理正确。

示例在本机商品 API 上加入 springdoc。数据来自内存，不连接数据库；接口没有配置认证，仅供学习。

## 完整例子

使用 JDK 17、Maven、Spring Boot 4.1.2 和 springdoc 3.1.0。不同 Boot 与 springdoc 大版本需核对兼容性。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>openapi-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.springdoc</groupId><artifactId>springdoc-openapi-starter-webmvc-ui</artifactId><version>3.1.0</version></dependency>
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
java -jar target/openapi-demo-1.0.jar --server.address=127.0.0.1 --server.port=8080
```

另开终端请求：

```powershell
curl.exe -i http://127.0.0.1:8080/v3/api-docs
curl.exe -i http://127.0.0.1:8080/products/1
```

第一条返回 OpenAPI JSON，`paths` 包含 `/products/{id}`；第二条返回商品 JSON。浏览器可访问 `http://127.0.0.1:8080/swagger-ui/index.html` 查看界面。结束后在服务窗口按 `Ctrl+C`；端口占用时同步修改地址。

## 工作中怎么用

为请求参数、状态码、错误响应、认证方式和字段约束维护准确描述，并用接口测试验证示例与实际行为。生成文档不等于服务端已经做输入校验，也不等于所有接口都可被任意用户调用。

文档和交互界面可能暴露内部路径或支持发起真实请求；生产环境应按项目的网络和授权策略决定是否开放。不要在示例或文档中写入真实令牌、密码或敏感数据。

## 动手与自查

1. 检查 `/v3/api-docs` 中的商品路径和 GET 方法，再分别请求存在与不存在的商品。
2. 修改接口路径后重新构建，对照文档与实际响应。
3. 为什么 Swagger UI 显示一个按钮，不能说明当前用户有权限调用该接口？

来源：[教程：集成Open API](https://liaoxuefeng.com/books/java/springboot/integration/open-api/index.html)、[springdoc](https://springdoc.org/)。
