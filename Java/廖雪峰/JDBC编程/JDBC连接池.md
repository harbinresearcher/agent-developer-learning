# JDBC连接池

> **重要程度：高**｜**手练：需要**
> **重点**：借用归还、容量、超时和连接生命周期。

## 连接池解决什么问题

建立数据库物理连接涉及网络、认证和服务器资源。连接池持有一定数量的连接，让业务借用和归还；应用通常面向 `DataSource` 获取连接。池大小受数据库连接上限和服务实例数共同约束，不是越大越快。

调用借出连接的 `close()` 通常是归还池，而不是关闭连接池本身。池应由应用容器管理并在应用结束时关闭。资源借用时间覆盖事务和查询处理，应尽量短，不能借到连接后长期等待外部服务。

## 完整例子

本例使用 JDK 11。将以下依赖加入 Maven 项目的 `pom.xml`：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.4.240</version>
</dependency>
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>7.0.2</version>
</dependency>
```

代码保存为 `src/main/java/JdbcPoolDemo.java`。在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$jdbcDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$jdbcDeps" JdbcPoolDemo
```

日志写入标准错误，以下只列标准输出。H2 是练习数据库，不代表真实数据库的方言和并发行为。

```java
import java.sql.*;
import java.util.logging.Logger;
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
public class JdbcPoolDemo {
    private static final Logger logger = Logger.getLogger(JdbcPoolDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：建立连接池
         * 数据源：H2 内存数据库
         * 操作：1) 配置地址和容量 2) 创建数据源
         * ======================================================================== */
        logger.info("开始创建连接池");
        // 1.1 设置练习用池参数
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:h2:mem:pooldemo");
        config.setMaximumPoolSize(2);
        config.setConnectionTimeout(1000);
        try (HikariDataSource dataSource = new HikariDataSource(config)) {
            logger.info("连接池创建完成");
            /* ========================================================================
             * 步骤2：借出连接并查询
             * 数据源：连接池提供的连接
             * 操作：1) 借用连接 2) 查询并归还
             * ======================================================================== */
            logger.info("开始借用连接");
            // 2.1 获取连接并读取结果
            try (Connection conn = dataSource.getConnection();
                 PreparedStatement ps = conn.prepareStatement("SELECT ? * ?")) {
                ps.setInt(1, 6); ps.setInt(2, 7);
                try (ResultSet rs = ps.executeQuery()) {
                    if (rs.next()) System.out.println("查询结果: " + rs.getInt(1));
                }
            }
            // 2.2 连接已经归还；离开外层作用域后关闭整个连接池
            logger.info("查询完成并归还连接");
        }
    }
}
```

标准输出：

```text
查询结果: 42
```

示例用 HikariCP 7.0.2 与 H2，只展示本地借用、执行、归还和关闭池。`maximumPoolSize=2` 只是练习值；`connectionTimeout=1000` 表示没有可借连接时最多等待约 1 秒，不能理解成 SQL 执行超时。缺少 SLF4J provider 时可能出现日志提示，不影响本例查询。

## 工作中怎么用

- 无论成功还是抛异常，都用 `try-with-resources` 关闭连接、语句和结果集。连接关闭与池关闭是两个不同生命周期。
- 容量需结合每个服务实例的池上限、实例数、数据库承载和其他客户端预算；监控等待时间、活跃/空闲连接数、超时和慢查询。
- 在事务中不要持有连接等待远程 HTTP；不要把同一个连接交给多个线程并发使用。借出连接的事务状态遵循池和框架契约，不要假设所有会话级设置都会自动恢复。
- 连接获取超时、查询超时和网络超时是不同配置。长连接在故障或空闲后如何检测和恢复，要结合数据库、驱动及部署环境验证。

## 动手与自查

1. 池设为 1，借出连接后暂不归还，再尝试借第二个，观察等待超时；实验后务必关闭第一个连接。
2. 在查询中故意抛出异常，确认 try-with-resources 仍归还连接。
3. 服务有 8 个实例，每个池最大 20 个连接，为什么不能直接把数据库上限设为 20？

来源：[教程：JDBC连接池](https://liaoxuefeng.com/books/java/jdbc/pool/index.html)、[HikariCP 配置](https://github.com/brettwooldridge/HikariCP)。
