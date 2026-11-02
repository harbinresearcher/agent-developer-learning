# 集成Hibernate

> **重要程度：高**｜**手练：需要**
> **重点**：实体映射、Session、持久化上下文、事务和查询数量。

## Hibernate 是什么

Hibernate 是常见的对象关系映射框架，也是 JPA 的一种实现。JPA 定义标准接口，Hibernate 提供实现并有自己的 `Session` API；Spring 可以管理实体管理器和事务。本例通过事务绑定的 `EntityManager` 获取 Hibernate `Session`，在隔离的 H2 内存库中保存和查询实体。

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
    <groupId>learn.example</groupId><artifactId>hibernate-demo</artifactId><version>1.0</version>
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

保存 `src/main/java/learn/jpa/HibernateDemo.java`：

```java
package learn.jpa;
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import org.hibernate.Session;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.orm.jpa.EntityManagerFactoryUtils;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.support.TransactionTemplate;
@SpringBootApplication
public class HibernateDemo {
    private static final Logger logger = LoggerFactory.getLogger(HibernateDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：在事务中保存并查询实体
         * 数据源：H2 内存数据库和 Hibernate Session
         * 操作：1) 保存商品 2) 在新事务中查询
         * ======================================================================== */
        logger.info("开始 Hibernate 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(HibernateDemo.class, args)) {
            EntityManagerFactory factory = context.getBean(EntityManagerFactory.class);
            TransactionTemplate transactions = new TransactionTemplate(context.getBean(PlatformTransactionManager.class));
            // 1.1 在事务中获取绑定的 EntityManager 并解包 Session
            Long id = transactions.execute(status -> {
                EntityManager manager = EntityManagerFactoryUtils.getTransactionalEntityManager(factory);
                if (manager == null) throw new IllegalStateException("事务实体管理器不可用");
                Session session = manager.unwrap(Session.class);
                Product product = new Product("keyboard");
                session.persist(product);
                session.flush();
                return product.getId();
            });
            // 1.2 新事务中按主键查找，取出需要的数据
            String name = transactions.execute(status -> {
                EntityManager manager = EntityManagerFactoryUtils.getTransactionalEntityManager(factory);
                if (manager == null) throw new IllegalStateException("事务实体管理器不可用");
                Product product = manager.unwrap(Session.class).find(Product.class, id);
                return product == null ? "未找到" : product.getName();
            });
            System.out.println("查询商品=" + name);
        }
        logger.info("Hibernate 示例完成");
    }
}
```

保存 `src/main/resources/application.properties`：

```properties
spring.datasource.url=jdbc:h2:mem:hibdemo;DB_CLOSE_ON_EXIT=FALSE
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.open-in-view=false
```

先确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/hibernate-demo-1.0.jar
```

日志中应看到 `查询商品=keyboard`。这里使用内存库，进程退出后数据消失；`create-drop` 只用于练习，会创建和删除表，生产环境应使用受控的数据库结构迁移。

## 看懂事务和 Session

第一个 `TransactionTemplate.execute` 开启事务，取得绑定到该事务的实体管理器，再解包出 Hibernate `Session`。`persist` 让新实体进入持久化上下文；`flush` 将待同步的更改发送到数据库，但不等于提交，事务正常结束后才提交。第二个事务根据主键查询，读取名称后返回。

示例中的实体管理器由 Spring 事务管理，不应随意手工关闭。真实项目通常可优先使用标准 JPA API，确有需要再使用 Hibernate 专有能力。版本应与项目中的 Spring、Hibernate 和 Jakarta API 匹配。

## 工作中注意

持久化上下文有自己的生命周期；实体脱离上下文后，未加载的延迟关联不一定能访问。遍历关联时可能发生 N+1 查询，应通过实际 SQL、查询数量和目标数据库执行计划排查。事务、锁和数据库约束仍需按业务设计，ORM 不自动解决并发更新。

不要无条件把实体直接作为 API 响应，避免暴露敏感字段或意外触发关联加载。大批量处理要考虑上下文大小、批次和内存，不能只看单条示例。

## 动手与自查

1. 修改保存的名称，验证新事务查询结果。
2. 在第一个事务 `flush` 后故意抛运行时异常，检查后续事务是否仍能查到记录。
3. 为什么 `flush` 成功不等于事务已经提交？

来源：[教程：集成Hibernate](https://liaoxuefeng.com/books/java/spring/database/hibernate/index.html)、[Hibernate ORM 用户指南](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)。
