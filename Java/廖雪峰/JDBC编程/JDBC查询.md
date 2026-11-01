# JDBC查询

> **重要程度：高**｜**手练：需要**
> **重点**：参数绑定、资源关闭和结果验证。

## 先理解查询

JDBC 是 Java 访问关系型数据库的接口；驱动负责与具体数据库通信。一次查询依次取得 `Connection`、创建 `PreparedStatement`、执行查询得到 `ResultSet`，从结果读取数据后关闭资源。

`?` 只代表一个**值**。`setString(1, value)` 的序号从 1 开始；驱动把数据和 SQL 语法分开处理，字符串中的引号不会变成 SQL 语法。表名、列名和排序方向不能用 `?` 代替；需要动态选择时应从固定白名单中选。

## 完整例子

本例使用 JDK 11 和 H2 2.4.240 内存数据库，不需要安装数据库服务。将下面依赖加入 Maven 项目的 `pom.xml`：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.4.240</version>
</dependency>
```

把代码保存为对应的 `src/main/java/类名.java`。在项目根目录用 `mvn package` 编译，再用 Maven 的依赖插件生成类路径：

```powershell
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$jdbcDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$jdbcDeps" JdbcQueryDemo
```

每次启动都是新数据库；示例仅证明 H2 的行为，生产库的 SQL 方言和事务行为应在目标数据库验证。日志写入标准错误，下面的输出只列标准输出。

```java
import java.sql.*;
import java.util.logging.Logger;

public class JdbcQueryDemo {
    private static final Logger logger = Logger.getLogger(JdbcQueryDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：准备查询数据
         * 数据源：当前进程的 H2 内存数据库
         * 操作：1) 建表 2) 插入包含引号的名称
         * ======================================================================== */
        logger.info("开始准备数据");
        try (Connection conn = DriverManager.getConnection("jdbc:h2:mem:query")) {
            // 1.1 创建临时表
            try (Statement st = conn.createStatement()) {
                st.executeUpdate("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100))");
            }
            // 1.2 绑定数据，避免把文本拼进 SQL
            try (PreparedStatement insert = conn.prepareStatement("INSERT INTO users VALUES (?, ?)")) {
                insert.setInt(1, 1);
                insert.setString(2, "O'Brien");
                insert.executeUpdate();
            }
            logger.info("准备数据完成");
            /* ========================================================================
             * 步骤2：按名称查询
             * 数据源：users 表
             * 操作：1) 绑定参数 2) 移动游标并读取列
             * ======================================================================== */
            logger.info("开始查询");
            // 2.1 绑定输入值
            try (PreparedStatement query = conn.prepareStatement("SELECT id, name FROM users WHERE name = ?")) {
                query.setString(1, "O'Brien");
                // 2.2 每次 next 移动一行，结果和语句自动关闭
                try (ResultSet rows = query.executeQuery()) {
                    while (rows.next()) {
                        System.out.println(rows.getInt("id") + ":" + rows.getString("name"));
                    }
                }
            }
            logger.info("查询完成");
        }
    }
}
```

标准输出：

```text
1:O'Brien
```

`executeQuery()` 返回结果集；它最初位于第一行之前。`next()` 返回 `true` 才能读当前行，返回 `false` 表示没有更多行。列名必须与查询结果匹配；重名列用 `AS` 起别名。SQL `NULL` 不能直接当成普通 Java 基本类型：例如 `getInt` 对 NULL 返回 0，随后要用 `wasNull()` 区分；也可按驱动支持使用 `getObject`。

## 工作中怎么用

- 用户输入一律按值绑定；不要为了模糊查询把输入直接拼到 SQL 中。`LIKE` 里 `%` 和 `_` 仍是通配符，按需求转义。
- 明确查询列，不把 `SELECT *` 当成稳定接口；大结果集要分页或按驱动能力流式处理，并检查执行计划和索引。
- 在 `try-with-resources` 内消费结果；不要返回一个依赖已关闭连接的 `ResultSet`。分页要有稳定排序；深分页和并发修改需要另行设计。

## 动手与自查

1. 改成查询不存在的名称，确认循环不执行。
2. 插入 `null` 并比较 `getInt`、`wasNull` 的结果：需要给表增加可空整数列。
3. 为什么 `WHERE name = ?` 可以绑定名称，而 `ORDER BY ?` 不能拿来选择列名？

来源：[教程：JDBC查询](https://liaoxuefeng.com/books/java/jdbc/query/index.html)、[Oracle JDBC 查询](https://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html)。
