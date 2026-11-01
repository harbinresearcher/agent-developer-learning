# 使用JDBC

> **重要程度：高**｜**手练：需要**
> **重点**：DataSource、JdbcTemplate、参数绑定、行映射和事务边界。

## Spring JDBC 解决什么问题

原生 JDBC 需要获取连接、创建语句、绑定参数、遍历结果和关闭资源。`JdbcTemplate` 封装这些重复步骤，业务代码仍要编写 SQL、决定参数并把每行结果映射成对象。它基于 `DataSource`，可以与 Spring 事务管理配合，但单独创建模板不等于开启多语句事务。

## 完整例子

使用 JDK 17、Maven、Spring Framework 7.0.10 和 H2 2.4.240。H2 是当前进程的内存数据库，不需要连接生产数据库。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>learn.example</groupId><artifactId>spring-ioc-demo</artifactId><version>1.0</version>
    <properties><maven.compiler.release>17</maven.compiler.release><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding></properties>
    <dependencies>
        <dependency><groupId>org.springframework</groupId><artifactId>spring-context</artifactId><version>7.0.10</version></dependency>
        <dependency><groupId>org.springframework</groupId><artifactId>spring-jdbc</artifactId><version>7.0.10</version></dependency>
        <dependency><groupId>com.h2database</groupId><artifactId>h2</artifactId><version>2.4.240</version></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

将代码保存为 `src/main/java/SpringJdbcDemo.java`：

```java
import java.util.List;
import java.util.logging.Logger;
import javax.sql.DataSource;
import org.h2.jdbcx.JdbcDataSource;
import org.springframework.jdbc.core.JdbcTemplate;
public class SpringJdbcDemo {
    private static final Logger logger = Logger.getLogger(SpringJdbcDemo.class.getName());
    record User(int id, String name) { }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：准备演示数据
         * 数据源：H2 内存数据库
         * 操作：1) 创建数据源和模板 2) 建表并插入记录
         * ======================================================================== */
        logger.info("开始准备数据");
        // 1.1 创建数据源，保留数据库直到当前 JVM 结束
        JdbcDataSource source = new JdbcDataSource();
        source.setURL("jdbc:h2:mem:springjdbc;DB_CLOSE_DELAY=-1");
        JdbcTemplate jdbc = new JdbcTemplate(source);
        // 1.2 创建表并绑定参数插入记录
        jdbc.execute("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50))");
        int inserted = jdbc.update("INSERT INTO users (id, name) VALUES (?, ?)", 1, "O'Brien");
        System.out.println("插入行数: " + inserted);
        logger.info("准备数据完成");
        /* ========================================================================
         * 步骤2：查询并映射结果
         * 数据源：users 表
         * 操作：1) 绑定名称 2) 逐行映射并输出
         * ======================================================================== */
        logger.info("开始查询用户");
        // 2.1 参数作为值传入，行映射器负责创建用户对象
        List<User> users = jdbc.query("SELECT id, name FROM users WHERE name = ?",
            (rs, rowNum) -> new User(rs.getInt("id"), rs.getString("name")), "O'Brien");
        // 2.2 消费已经映射的结果
        for (User user : users) System.out.println(user.id() + ":" + user.name());
        logger.info("用户查询完成");
    }
}
```

先运行 `java -version` 和 `mvn -v`，确认两者使用 JDK 17 或以上。PowerShell 在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" SpringJdbcDemo
```

标准输出（日志单独显示）：

```text
插入行数: 1
1:O'Brien
```

## 看懂执行过程

`JdbcDataSource` 提供 H2 连接；它是数据源，不是本例的连接池。`JdbcTemplate.update` 返回更新行数；`?` 参数由驱动绑定，不拼进 SQL。`query` 对每一行调用 `RowMapper`，创建 `User` 列表；没有匹配行时该列表为空。例子用 `record` 保存结果，要求 Java 16 以上，本例统一使用 Java 17。

`DB_CLOSE_DELAY=-1` 只为让示例数据库在模板关闭短连接后继续存在，JVM 退出时演示数据消失。不同数据库方言和连接行为需要在目标数据库验证。

## 工作中怎么用

- 用户输入按值绑定；表名、列名和排序方向不能用参数占位符代替，动态选择应使用白名单。
- 大结果集不应一律读入 `List`；按业务分页或使用合适的流式 API，并检查驱动和连接生命周期。
- Spring 会将数据库访问异常转换为 `DataAccessException` 层次；不要为了隐藏异常而直接返回空结果。`queryForObject` 对结果行数有要求，不等同于“查不到就返回 null”。
- 生产通常注入由容器管理的连接池 `DataSource` 和 `JdbcTemplate`。多步一致性使用匹配同一数据源的事务管理器，参见 [[Java/廖雪峰/Spring开发/访问数据库/使用声明式事务|声明式事务]]。

## 动手与自查

1. 查询不存在的名称，验证结果列表为空；再比较单行查询的行为。
2. 插入带单引号的名称，说明参数绑定如何避免把它当作 SQL 语法。
3. 用模板连续更新两张表时，为什么还要明确事务边界？

来源：[教程：使用JDBC](https://liaoxuefeng.com/books/java/spring/database/jdbc/index.html)、[Spring JdbcTemplate](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html)。
