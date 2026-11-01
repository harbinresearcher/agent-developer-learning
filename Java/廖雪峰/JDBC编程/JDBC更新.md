# JDBC更新

> **重要程度：高**｜**手练：需要**
> **重点**：参数化更新、影响行数及事务边界。

## 更新数据的基本步骤

`INSERT`、`UPDATE` 和 `DELETE` 通常用 `PreparedStatement.executeUpdate()`，返回受影响行数。先写带 `?` 的 SQL，再按从 1 开始的位置绑定值；不要把输入拼到 SQL 字符串中。更新不存在的行常返回 0，但具体受影响行数语义应以数据库和驱动配置为准。

## 完整例子

本例使用 JDK 11。将以下依赖加入 Maven 项目的 `pom.xml`：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.4.240</version>
</dependency>
```

代码保存为 `src/main/java/JdbcUpdateDemo.java`。在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$jdbcDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$jdbcDeps" JdbcUpdateDemo
```

日志写入标准错误，以下只列标准输出。H2 是练习数据库，不代表真实数据库的方言和并发行为。

```java
import java.sql.*;
import java.util.logging.Logger;
public class JdbcUpdateDemo {
    private static final Logger logger = Logger.getLogger(JdbcUpdateDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：创建和插入
         * 数据源：H2 内存数据库
         * 操作：1) 建表 2) 插入一条记录
         * ======================================================================== */
        logger.info("开始准备数据");
        try (Connection conn = DriverManager.getConnection("jdbc:h2:mem:update")) {
            // 1.1 创建表
            try (Statement st = conn.createStatement()) {
                st.executeUpdate("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50))");
            }
            // 1.2 插入绑定参数
            try (PreparedStatement ps = conn.prepareStatement("INSERT INTO users VALUES (?, ?)")) {
                ps.setInt(1, 1); ps.setString(2, "Ada");
                System.out.println("插入: " + ps.executeUpdate());
            }
            logger.info("准备数据完成");
            /* ========================================================================
             * 步骤2：更新并检查行数
             * 数据源：users 表
             * 操作：1) 更新已存在记录 2) 更新不存在记录
             * ======================================================================== */
            logger.info("开始更新");
            // 2.1 每次绑定名称和 id
            try (PreparedStatement ps = conn.prepareStatement("UPDATE users SET name = ? WHERE id = ?")) {
                ps.setString(1, "Bob"); ps.setInt(2, 1);
                System.out.println("更新存在的行: " + ps.executeUpdate());
                ps.setString(1, "Nobody"); ps.setInt(2, 99);
                System.out.println("更新不存在的行: " + ps.executeUpdate());
            }
            logger.info("更新完成");
            /* ========================================================================
             * 步骤3：核对最终数据
             * 数据源：users 表
             * 操作：1) 查询名称 2) 输出结果
             * ======================================================================== */
            logger.info("开始核对");
            // 3.1 查询修改结果
            try (Statement st = conn.createStatement(); ResultSet rs = st.executeQuery("SELECT name FROM users WHERE id = 1")) {
                if (rs.next()) System.out.println("当前名称: " + rs.getString(1));
            }
            logger.info("核对完成");
        }
    }
}
```

标准输出：

```text
插入: 1
更新存在的行: 1
更新不存在的行: 0
当前名称: Bob
```

插入和更新使用不同语句；更新对象由 `WHERE id = ?` 限定。忘记 `WHERE` 可能修改整张表。检查返回行数可以识别目标不存在，但某些数据库和驱动对“匹配行”和“实际变更行”的计数规则不同，不能脱离实际配置推断。

## 工作中怎么用

- 修改前明确目标条件；涉及余额、库存等并发数据，条件更新或乐观锁通常需要检查行数。
- 参数只能绑定值；动态表名和排序列使用固定白名单。数据库约束负责兜底，参数化并不能代替唯一约束和业务校验。
- 本例使用连接默认自动提交，多条需要一起成功的更新应使用明确事务。需要数据库生成主键时可请求 `RETURN_GENERATED_KEYS` 并读取 `getGeneratedKeys()`，具体支持和返回列取决于驱动。

## 动手与自查

1. 增加 `DELETE FROM users WHERE id = ?`，先删除存在的 id，再删除一次，比较返回值。
2. 在隔离测试库验证漏写 `WHERE` 的影响，再说明怎样降低生产误操作风险。
3. 为库存扣减设计 `WHERE stock >= ?` 条件，解释 0 行如何影响业务返回。

来源：[教程：JDBC更新](https://liaoxuefeng.com/books/java/jdbc/update/index.html)、[Oracle PreparedStatement](https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html)。
