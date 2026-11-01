# JDBC批处理

> **重要程度：高**｜**手练：需要**
> **重点**：参数绑定、资源关闭和结果验证。

## 批处理是什么

同一条 SQL 要处理多组参数时，逐组绑定后调用 `addBatch()`，再调用 `executeBatch()` 执行。它减少应用与驱动的调用次数；是否合并网络往返、重写为多行 SQL 取决于驱动和配置，不能保证必然更快。

`executeBatch()` 与事务是两回事：前者组织待执行语句，后者决定提交和回滚范围。示例关闭自动提交，并在成功后提交，失败时回滚。

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
java -cp "target/classes;$jdbcDeps" JdbcBatchDemo
```

每次启动都是新数据库；示例仅证明 H2 的行为，生产库的 SQL 方言和事务行为应在目标数据库验证。日志写入标准错误，下面的输出只列标准输出。

```java
import java.sql.*;
import java.util.Arrays;
import java.util.logging.Logger;

public class JdbcBatchDemo {
    private static final Logger logger = Logger.getLogger(JdbcBatchDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：准备数据表
         * 数据源：H2 内存数据库
         * 操作：1) 创建连接 2) 创建空表
         * ======================================================================== */
        logger.info("开始准备表");
        try (Connection conn = DriverManager.getConnection("jdbc:h2:mem:batch")) {
            // 1.1 创建用于演示的表
            try (Statement st = conn.createStatement()) {
                st.executeUpdate("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50))");
            }
            logger.info("准备表完成");
            /* ========================================================================
             * 步骤2：批量插入
             * 数据源：三条待插入记录
             * 操作：1) 逐条绑定并加入批次 2) 提交或回滚整个事务
             * ======================================================================== */
            logger.info("开始批量插入");
            conn.setAutoCommit(false);
            try (PreparedStatement ps = conn.prepareStatement("INSERT INTO users VALUES (?, ?)")) {
                // 2.1 每次绑定一条记录，然后加入批次
                for (int i = 1; i <= 3; i++) {
                    ps.setInt(1, i);
                    ps.setString(2, "user" + i);
                    ps.addBatch();
                }
                // 2.2 一次执行批次，成功后提交
                int[] counts = ps.executeBatch();
                conn.commit();
                System.out.println("影响行数: " + Arrays.toString(counts));
            } catch (SQLException ex) {
                conn.rollback();
                throw ex;
            }
            logger.info("批量插入完成");
            /* ========================================================================
             * 步骤3：核对结果
             * 数据源：提交后的 users 表
             * 操作：1) 查询记录数 2) 输出数量
             * ======================================================================== */
            logger.info("开始核对记录数");
            // 3.1 查询并读取聚合结果
            try (Statement st = conn.createStatement(); ResultSet rs = st.executeQuery("SELECT COUNT(*) FROM users")) {
                rs.next();
                System.out.println("记录数: " + rs.getInt(1));
            }
            logger.info("核对记录数完成");
        }
    }
}
```

标准输出：

```text
影响行数: [1, 1, 1]
记录数: 3
```

本例 H2 为每次插入返回 1。JDBC 驱动也可能返回 `Statement.SUCCESS_NO_INFO`（成功但具体行数未知）；批次失败可能抛出 `BatchUpdateException`，其更新计数是否包含失败后的条目取决于驱动是否继续执行。不要把所有负数都当成成功，也不要假设失败时数据库一定没有执行任何语句。

## 工作中怎么用

- 先定义失败策略：整批回滚，还是按业务要求分块重试。示例对同一连接上的批次执行失败回滚；真实系统还要处理回滚或提交时连接中断带来的结果不确定。
- 大批数据分块，避免堆积参数占用内存或产生过长事务。块大小应依据驱动、数据库、网络和业务压测，不存在通用最佳值。
- 重试前考虑唯一约束和幂等；失败后再次提交整批不能假设绝对安全。日志不要输出敏感参数。

## 动手与自查

1. 将循环中的第三个 id 改为 1，观察批处理异常和回滚；可在 catch 中查询并确认表为空。
2. 试着每两条执行一个批次，说明提交放在循环内外的区别。
3. `addBatch()` 后为什么还要显式 `commit()`？

来源：[教程：JDBC批处理](https://liaoxuefeng.com/books/java/jdbc/batch/index.html)、[Oracle JDBC Statement](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/Statement.html#executeBatch())。
