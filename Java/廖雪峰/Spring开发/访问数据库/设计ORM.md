# 设计ORM

> **重要程度：中**｜**手练：需要**
> **重点**：字段元数据、SQL 占位符、反射、绑定顺序和安全边界。

## ORM 的基本工作

对象关系映射把 Java 对象的字段与关系表的列联系起来。框架读取映射元数据，将对象值绑定到 SQL 参数，再将查询结果映射回对象。本例只实现一个受限的插入演示，不是完整 ORM。

## 完整例子

使用 JDK 17、Maven 和 H2 2.4.240。新建目录，保存 `pom.xml`：

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

保存 `src/main/java/OrmDemo.java`：

```java
import java.lang.annotation.*;
import java.lang.reflect.Field;
import java.sql.*;
import java.util.*;
import java.util.logging.Logger;
public class OrmDemo {
    private static final Logger logger = Logger.getLogger(OrmDemo.class.getName());
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.FIELD)
    @interface Column { String value(); }
    static class User {
        @Column("user_name") public final String name;
        @Column("age") public final int age;
        User(String name, int age) { this.name = name; this.age = age; }
    }
    static int insert(Connection connection, User user) throws Exception {
        List<Field> fields = new ArrayList<>();
        for (Field field : User.class.getFields()) {
            if (field.isAnnotationPresent(Column.class)) fields.add(field);
        }
        fields.sort(Comparator.comparing(field -> field.getAnnotation(Column.class).value()));
        List<String> columns = new ArrayList<>();
        List<String> placeholders = new ArrayList<>();
        for (Field field : fields) {
            String column = field.getAnnotation(Column.class).value();
            if (!Set.of("age", "user_name").contains(column)) throw new IllegalArgumentException("列名不允许");
            columns.add(column);
            placeholders.add("?");
        }
        String sql = "INSERT INTO users (" + String.join(", ", columns) + ") VALUES (" + String.join(", ", placeholders) + ")";
        try (PreparedStatement statement = connection.prepareStatement(sql)) {
            for (int i = 0; i < fields.size(); i++) statement.setObject(i + 1, fields.get(i).get(user));
            return statement.executeUpdate();
        }
    }
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：演示字段映射与参数绑定
         * 数据源：H2 内存数据库和注解字段
         * 操作：1) 建表并插入 2) 查询并核对
         * ======================================================================== */
        logger.info("开始 ORM 映射示例");
        try (Connection connection = DriverManager.getConnection("jdbc:h2:mem:orm")) {
            // 1.1 创建演示表，按注解生成参数化插入
            try (Statement statement = connection.createStatement()) {
                statement.executeUpdate("CREATE TABLE users (user_name VARCHAR(100), age INT)");
            }
            System.out.println("插入=" + insert(connection, new User("Alice", 18)));
            // 1.2 查询真实数据库中的结果
            try (Statement statement = connection.createStatement();
                 ResultSet rows = statement.executeQuery("SELECT user_name, age FROM users")) {
                if (rows.next()) System.out.println("用户=" + rows.getString(1) + "," + rows.getInt(2));
            }
        }
        logger.info("ORM 映射示例完成");
    }
}
```

先确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上。在工程目录的 PowerShell 中运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$ormDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$ormDeps" OrmDemo
```

标准输出，日志单独显示：

```text
插入=1
用户=Alice,18
```

## 从对象到数据库

`@Column` 在运行时保留字段到列名的映射。`insert` 读取 `User` 的公开注解字段，按列名排序，再生成对应数量的 `?`；排序后的同一字段列表也用于绑定值，因此列和参数顺序一致。表名固定，列名只允许两个预先定义的值；用户值通过 `PreparedStatement` 绑定，不拼入 SQL。

查询部分仍手工读取 `ResultSet`，并没有实现自动对象构造。示例仅支持一个固定实体和表，不处理主键生成、关系、空值类型、更新、批处理或不同数据库方言，不能直接作为生产 ORM。

## 工作中需要关注

真实框架还要处理事务、持久化上下文、并发、关联加载、缓存和异常。映射不消除 SQL 与索引成本；应观察实际查询和执行计划。反射访问私有成员还会受到模块与访问权限限制。

动态标识符必须来自可信定义或白名单；参数绑定保护的是值，不能把任意表名或列名放进占位符。生产项目优先使用成熟框架，并通过迁移工具管理数据库结构。

## 动手与自查

1. 增加一个带引号的名称，验证它作为值写入数据库。
2. 打印生成的 SQL，检查列顺序与绑定顺序是否一致。
3. 新增不在白名单的列注解，观察异常；解释为什么不能直接信任外部传入的列名。

来源：[教程：设计ORM](https://liaoxuefeng.com/books/java/spring/database/orm/index.html)、[JDBC PreparedStatement](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/PreparedStatement.html)。
