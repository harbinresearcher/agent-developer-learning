# JDBC事务

> **重要程度：高**｜**手练：需要**
> **重点**：参数绑定、资源关闭和结果验证。

## 事务解决什么问题

转账需要扣款和收款都成功。事务把同一数据库连接上的多个操作组成一个提交单位：`commit()` 确认，`rollback()` 撤销尚未提交的修改。JDBC 连接通常默认自动提交；每条语句成功后即可提交，因此多条关联修改要显式开启事务或交给框架管理。

## 完整例子：失败后回滚

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
java -cp "target/classes;$jdbcDeps" JdbcTransactionDemo
```

每次启动都是新数据库；示例仅证明 H2 的行为，生产库的 SQL 方言和事务行为应在目标数据库验证。日志写入标准错误，下面的输出只列标准输出。

```java
import java.sql.*;
import java.util.logging.Logger;

public class JdbcTransactionDemo {
    private static final Logger logger = Logger.getLogger(JdbcTransactionDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：准备账户
         * 数据源：H2 内存数据库
         * 操作：1) 建表 2) 写入初始余额
         * ======================================================================== */
        logger.info("开始准备账户");
        try (Connection conn = DriverManager.getConnection("jdbc:h2:mem:tx")) {
            // 1.1 建表与初始化在事务演示前完成
            try (Statement st = conn.createStatement()) {
                st.executeUpdate("CREATE TABLE accounts (id INT PRIMARY KEY, balance INT NOT NULL)");
                st.executeUpdate("INSERT INTO accounts VALUES (1, 100), (2, 50)");
            }
            logger.info("账户准备完成");
            /* ========================================================================
             * 步骤2：演示失败回滚
             * 数据源：两个账户
             * 操作：1) 扣款并模拟失败 2) 捕获异常后回滚
             * ======================================================================== */
            logger.info("开始模拟转账");
            // 2.1 手动控制这条连接的事务
            conn.setAutoCommit(false);
            try {
                try (PreparedStatement debit = conn.prepareStatement(
                        "UPDATE accounts SET balance = balance - ? WHERE id = ?")) {
                    debit.setInt(1, 30);
                    debit.setInt(2, 1);
                    if (debit.executeUpdate() != 1) throw new SQLException("账户不存在");
                }
                // 2.2 假设收款步骤失败，验证扣款能否撤销
                throw new SQLException("模拟收款失败");
            } catch (SQLException ex) {
                conn.rollback();
                System.out.println("已回滚: " + ex.getMessage());
            }
            logger.info("模拟转账完成");
            /* ========================================================================
             * 步骤3：核对余额
             * 数据源：回滚后的 accounts 表
             * 操作：1) 查询两行 2) 按账户顺序输出
             * ======================================================================== */
            logger.info("开始核对余额");
            // 3.1 读取回滚后的余额
            try (Statement st = conn.createStatement();
                 ResultSet rows = st.executeQuery("SELECT id, balance FROM accounts ORDER BY id")) {
                while (rows.next()) System.out.println(rows.getInt(1) + ":" + rows.getInt(2));
            }
            logger.info("核对余额完成");
        }
    }
}
```

标准输出：

```text
已回滚: 模拟收款失败
1:100
2:50
```

扣款执行后故意抛出异常；`rollback()` 撤销扣款，两个账户余额仍是初始值。示例只展示失败路径。成功路径需要在扣款和收款均成功后调用 `conn.commit()`；关闭连接不能代替明确提交。建表放在事务开始前，是为了避免某些数据库的 DDL 隐式提交影响演示。

## 工作中怎么用

- 事务边界包住必须一起成功的数据库操作，检查每条更新的影响行数。事务不是余额并发安全的替代品：还要根据业务用条件更新、锁或版本号。
- 隔离级别控制并发事务可见性，具体表现由数据库决定；不要只记名称，需在目标数据库验证。异常时回滚也可能失败，应保留原始异常并记录回滚失败；生产代码通常交给成熟事务管理器处理。
- 数据库事务不能自动撤销 HTTP 请求或已经发出的消息。跨系统一致性另需事务消息、补偿或其他明确方案。
- 连接池归还连接时遵循池和框架的状态恢复约定；手工管理时避免泄漏自动提交、隔离级别等状态。

## 动手与自查

1. 删除模拟异常，增加账户 2 的收款更新并 `commit()`，结果应为 70、80。
2. 再让收款更新影响 0 行并抛异常，确认余额回到 100、50。
3. 两个连接同时转出时，只靠 `commit()` 为什么不能保证余额检查安全？

来源：[教程：JDBC事务](https://liaoxuefeng.com/books/java/jdbc/tx/index.html)、[Oracle JDBC 事务](https://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html)。
