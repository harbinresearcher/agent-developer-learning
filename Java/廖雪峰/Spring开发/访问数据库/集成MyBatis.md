# 集成MyBatis

> **重要程度：高**｜**手练：需要**
> **重点**：Mapper、参数绑定、结果映射、SQL 注入和事务边界。

## MyBatis 做什么

MyBatis 将 SQL 和 Java 方法、参数及结果映射关联起来。开发者仍需设计 SQL、索引和事务；它不是不需要理解数据库的自动优化器。本例用注解声明 Mapper 查询，Spring Boot 负责发现 Mapper 和数据源；H2 内存数据库用于隔离练习。

## 完整例子

使用 JDK 17、Maven、Spring Boot 4.1.2 与 MyBatis Spring Boot Starter 4.0.0。Starter 4.0 对应 Spring Boot 4.0 及以上，旧项目应核对自己的版本组合。新建目录并保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>mybatis-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
        <dependency><groupId>org.mybatis.spring.boot</groupId><artifactId>mybatis-spring-boot-starter</artifactId><version>4.0.0</version></dependency>
        <dependency><groupId>com.h2database</groupId><artifactId>h2</artifactId><scope>runtime</scope></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/mybatis/UserMapper.java`：

```java
package learn.mybatis;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
@Mapper
public interface UserMapper {
    @Select("SELECT name FROM users WHERE id = #{id}")
    String findName(@Param("id") int id);
    @Select("SELECT id FROM users WHERE name = #{name}")
    Integer findId(@Param("name") String name);
}
```

保存 `src/main/java/learn/mybatis/MybatisDemo.java`：

```java
package learn.mybatis;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;
@SpringBootApplication
public class MybatisDemo {
    private static final Logger logger = LoggerFactory.getLogger(MybatisDemo.class);
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：准备并查询测试用户
         * 数据源：H2 内存数据库和 MyBatis Mapper
         * 操作：1) 建表并插入 2) 按参数查询
         * ======================================================================== */
        logger.info("开始 MyBatis 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(MybatisDemo.class, args)) {
            // 1.1 创建测试表并插入带引号的名称
            JdbcTemplate jdbc = context.getBean(JdbcTemplate.class);
            jdbc.execute("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100) NOT NULL)");
            jdbc.update("INSERT INTO users VALUES (?, ?)", 1, "O'Brien");
            // 1.2 调用 Mapper 查询存在和不存在的记录
            UserMapper mapper = context.getBean(UserMapper.class);
            System.out.println("名称=" + mapper.findName(1));
            System.out.println("编号=" + mapper.findId("O'Brien"));
            System.out.println("无结果=" + (mapper.findName(99) == null));
        }
        logger.info("MyBatis 示例完成");
    }
}
```

先用 `java -version` 和 `mvn -v` 确认使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/mybatis-demo-1.0.jar
```

日志中应看到：

```text
名称=O'Brien
编号=1
无结果=true
```

`@Mapper` 标记接口，Starter 在符合条件时注册 Mapper；`@Select` 声明查询，`@Param` 给参数命名。`#{name}` 作为预处理语句的值绑定，引号属于数据。本例的单值查询没有匹配记录时返回 null；多行结果和对象映射应按方法契约设计。

## 工作中怎么用

`#{...}` 用于绑定值；`${...}` 是文本替换，不能直接接收不可信输入。表名、列名和排序方向不能简单当作值绑定，动态选择时应使用固定白名单。不要把名称中的引号手工拼到 SQL 中。

复杂查询应明确列名、映射、分页和索引，并在目标数据库检查执行计划。XML 映射适合较复杂 SQL，但命名空间、方法名和参数需要与 Mapper 对应。跨多个数据库操作要明确 Spring 事务边界和回滚规则；本例只在内存数据库验证基础查询。

## 动手与自查

1. 查询不存在的 ID，检查返回值；再测试包含引号的名称。
2. 新增一个按名称查询列表的方法，验证无结果和多结果。
3. 为什么 `${name}` 不能代替 `#{name}` 来安全绑定不可信字符串？

来源：[教程：集成MyBatis](https://liaoxuefeng.com/books/java/spring/database/mybatis/index.html)、[MyBatis Spring Boot Starter](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)。
