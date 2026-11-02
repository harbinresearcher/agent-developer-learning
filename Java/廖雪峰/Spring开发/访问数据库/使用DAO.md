# 使用DAO

> **重要程度：高**｜**手练：需要**
> **重点**：数据访问边界、参数化 SQL、结果映射和事务职责。

## DAO 是什么

DAO（数据访问对象）集中处理某类数据的读取和写入。业务服务通过明确的方法访问数据，不必在各处重复拼写 SQL 和结果映射。例子中的 `UserDao` 使用 `JdbcTemplate`，提供插入和按主键查询；它不负责 HTTP 响应或整个业务流程。

## 完整例子

使用 JDK 17、Maven、Spring Framework 7.0.10 与 H2 2.4.240。新建目录并保存 `pom.xml`：

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

保存 `src/main/java/DaoDemo.java`：

```java
import java.util.List;
import java.util.Optional;
import java.util.logging.Logger;
import org.h2.jdbcx.JdbcDataSource;
import org.springframework.jdbc.core.JdbcTemplate;
public class DaoDemo {
    private static final Logger logger = Logger.getLogger(DaoDemo.class.getName());
    static class User {
        final int id;
        final String name;
        User(int id, String name) { this.id = id; this.name = name; }
    }
    static class UserDao {
        private final JdbcTemplate jdbc;
        UserDao(JdbcTemplate jdbc) { this.jdbc = jdbc; }
        int insert(User user) {
            return jdbc.update("INSERT INTO users (id, name) VALUES (?, ?)", user.id, user.name);
        }
        Optional<User> findById(int id) {
            List<User> rows = jdbc.query("SELECT id, name FROM users WHERE id = ?",
                (rs, row) -> new User(rs.getInt("id"), rs.getString("name")), id);
            return rows.stream().findFirst();
        }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：准备并访问用户数据
         * 数据源：H2 内存数据库
         * 操作：1) 创建表和 DAO 2) 插入并查询
         * ======================================================================== */
        logger.info("开始 DAO 示例");
        // 1.1 初始化隔离的演示数据库
        JdbcDataSource source = new JdbcDataSource();
        source.setURL("jdbc:h2:mem:daodemo;DB_CLOSE_DELAY=-1");
        JdbcTemplate jdbc = new JdbcTemplate(source);
        jdbc.execute("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100) NOT NULL)");
        UserDao dao = new UserDao(jdbc);
        // 1.2 保存用户并检查存在及不存在的记录
        System.out.println("插入=" + dao.insert(new User(1, "Alice")));
        System.out.println("名称=" + dao.findById(1).orElseThrow().name);
        System.out.println("不存在=" + dao.findById(2).isEmpty());
        logger.info("DAO 示例完成");
    }
}
```

先用 `java -version` 和 `mvn -v` 确认 JDK 17 或以上。在工程目录的 PowerShell 中运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" DaoDemo
```

标准输出，日志单独显示：

```text
插入=1
名称=Alice
不存在=true
```

`insert` 通过占位符绑定值并返回影响行数；`findById` 将结果行映射为 `User`，没有记录时返回空 `Optional`。表的主键约束保证本例按 ID 查询最多一行。H2 内存库只用于练习，JVM 结束后数据消失。

## 工作中怎么用

DAO 封装 SQL 不等于不需要理解 SQL：仍要检查索引、分页、约束、结果行数和目标数据库方言。参数绑定用于值，动态表名和排序列仍需白名单。

跨多个 DAO 的业务操作需要明确事务边界，通常由服务层或事务管理器协调；一次调用成功不代表整个流程自动原子。生产项目一般由容器管理连接池和 DAO，测试既要覆盖无记录和错误，也要在目标数据库验证关键查询。

## 动手与自查

1. 查找不存在的 ID，解释为什么返回空 `Optional`。
2. 增加按 ID 更新名称的方法，检查 1 行和 0 行两种结果。
3. 两个 DAO 更新需要一起成功时，事务边界为什么不能只凭两个方法名推断？

来源：[教程：使用DAO](https://liaoxuefeng.com/books/java/spring/database/dao/index.html)、[Spring JDBC](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html)。
