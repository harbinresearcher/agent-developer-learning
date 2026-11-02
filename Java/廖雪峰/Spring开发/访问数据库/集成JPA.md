# 集成JPA

> **重要程度：高**｜**手练：需要**
> **重点**：实体映射、主键生成、仓储查询、事务与持久化上下文。

## JPA、Hibernate 和 Spring Data 的关系

JPA 是对象关系映射的规范；Hibernate 是常见实现；Spring Data JPA 在其上提供仓储接口等便利。本例用实体映射 `products` 表，使用 `JpaRepository` 保存并按主键查询，底层使用隔离的 H2 内存数据库。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>jpa-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-data-jpa</artifactId></dependency>
        <dependency><groupId>com.h2database</groupId><artifactId>h2</artifactId><scope>runtime</scope></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/jpa/Product.java`：

```java
package learn.jpa;
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
@Entity
@Table(name = "products")
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false, length = 100)
    private String name;
    protected Product() { }
    public Product(String name) { this.name = name; }
    public Long getId() { return id; }
    public String getName() { return name; }
}
```

保存 `src/main/java/learn/jpa/ProductRepository.java`：

```java
package learn.jpa;
import org.springframework.data.jpa.repository.JpaRepository;
public interface ProductRepository extends JpaRepository<Product, Long> { }
```

保存 `src/main/java/learn/jpa/JpaDemo.java`：

```java
package learn.jpa;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
@SpringBootApplication
public class JpaDemo {
    private static final Logger logger = LoggerFactory.getLogger(JpaDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：保存并查询商品
         * 数据源：启动时创建的 H2 内存数据库
         * 操作：1) 保存实体 2) 根据生成的主键查询
         * ======================================================================== */
        logger.info("开始 JPA 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(JpaDemo.class, args)) {
            // 1.1 获取容器管理的仓储并保存对象
            ProductRepository repository = context.getBean(ProductRepository.class);
            Product saved = repository.save(new Product("keyboard"));
            // 1.2 根据生成的标识查找并验证不存在的记录
            Product found = repository.findById(saved.getId()).orElseThrow();
            System.out.println("查询商品=" + found.getName());
            System.out.println("不存在=" + repository.findById(-1L).isEmpty());
        }
        logger.info("JPA 示例结束");
    }
}
```

保存 `src/main/resources/application.properties`：

```properties
spring.datasource.url=jdbc:h2:mem:jpademo;DB_CLOSE_ON_EXIT=FALSE
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.open-in-view=false
```

确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/jpa-demo-1.0.jar
```

日志中应看到：

```text
查询商品=keyboard
不存在=true
```

`@Entity` 声明实体，`@Id` 标识主键，`IDENTITY` 由数据库生成标识；受保护的无参构造器供持久化框架使用。仓储的 `save` 与 `findById` 由 Spring Data 提供，查询不到时返回空 `Optional`。示例使用内存库，进程结束后数据消失。

## 事务与工作边界

实体通常在持久化上下文中被管理；修改受管理实体可能在刷新时同步到数据库，具体时机受事务和提供者影响。延迟加载依赖仍有效的持久化上下文；不能假设事务结束后所有关联都可访问。避免在循环中意外触发大量查询，需结合 SQL 日志与查询计划排查 N+1。

示例设置 `ddl-auto=create-drop` 方便练习，会创建并删除表，不能直接用于生产数据库。生产环境通常使用受控的数据库迁移工具管理结构变更。实体不应无条件作为 API 响应，防止敏感字段暴露及意外加载关联；复杂查询、分页和并发更新都需要在目标数据库验证。

## 动手与自查

1. 增加一个商品并查询，确认生成的 ID 和名称。
2. 查询不存在的主键，比较 `isEmpty()` 与 `orElseThrow()` 的行为。
3. 解释 JPA 规范、Hibernate 实现和 Spring Data 仓储各自负责什么。

来源：[教程：集成JPA](https://liaoxuefeng.com/books/java/spring/database/jpa/index.html)、[Spring Boot SQL 数据访问](https://docs.spring.io/spring-boot/reference/data/sql.html)。
