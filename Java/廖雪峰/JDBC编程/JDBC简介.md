# JDBC简介

> **重要程度：高**｜**手练：需要**
> **重点**：驱动、连接、语句、结果集和资源关闭。

## 先建立整体认识

JDBC 是 Java 访问关系型数据库的一组标准接口，驱动是数据库厂商或第三方提供的实现。应用先获得连接 `Connection`，用它创建语句 `Statement` / `PreparedStatement`；查询时通过 `ResultSet` 读取结果，结束后依次关闭结果、语句和连接。

连接地址（JDBC URL）、驱动和账号依数据库而异。现代驱动一般通过服务发现自动注册；具体以驱动文档为准。示例用 H2 内存数据库，每次启动创建全新数据，不访问真实服务器。

## 完整例子

本例使用 JDK 11。将以下依赖加入 Maven 项目的 `pom.xml`：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.4.240</version>
</dependency>
```

代码保存为 `src/main/java/JdbcIntroDemo.java`。在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$jdbcDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$jdbcDeps" JdbcIntroDemo
```

日志写入标准错误，以下只列标准输出。H2 是练习数据库，不代表真实数据库的方言和并发行为。

```java
import java.sql.*;
import java.util.logging.Logger;
public class JdbcIntroDemo {
    private static final Logger logger = Logger.getLogger(JdbcIntroDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：建立连接
         * 数据源：H2 内存数据库
         * 操作：1) 根据 URL 获取连接 2) 验证连接状态
         * ======================================================================== */
        logger.info("开始建立连接");
        // 1.1 创建临时连接
        try (Connection conn = DriverManager.getConnection("jdbc:h2:mem:intro")) {
            System.out.println("连接有效: " + conn.isValid(1));
            logger.info("连接建立完成");
            /* ========================================================================
             * 步骤2：执行查询
             * 数据源：数据库表达式
             * 操作：1) 发送 SQL 2) 读取结果
             * ======================================================================== */
            logger.info("开始执行查询");
            // 2.1 创建语句和结果集，离开作用域后自动关闭
            try (Statement st = conn.createStatement(); ResultSet rs = st.executeQuery("SELECT 6 * 7")) {
                // 2.2 移动到第一行后读取第一列
                if (rs.next()) System.out.println("数据库乘法结果: " + rs.getInt(1));
            }
            logger.info("查询完成");
        }
    }
}
```

标准输出：

```text
连接有效: true
数据库乘法结果: 42
```

`DriverManager.getConnection` 根据 URL 选择驱动并获取连接。常量 SQL 用 `Statement` 足够；带用户输入时用 `PreparedStatement` 的参数绑定。`try-with-resources` 按声明的相反顺序关闭资源，异常时也会关闭。

## 工作中怎么用

生产系统一般由连接池提供 `DataSource`，请求结束后关闭借出的连接以归还池，而不是每次新建物理连接。数据库地址、账号和密码放在环境配置或密钥管理中，不能硬编码或写进日志。

连接建立可能因地址、权限、驱动、网络或数据库不可用而失败；先识别异常并结合错误码定位。不要记录完整敏感连接串。使用 ORM 或 Spring JDBC 时，底层仍涉及连接、SQL、事务和资源生命周期。

## 动手与自查

1. 将 SQL 改为 `SELECT 3 + 4`，预测并验证输出。
2. 如果关闭连接后继续读取 `ResultSet`，为什么不可靠？
3. 数据库驱动与 JDBC 接口各负责什么？

来源：[教程：JDBC简介](https://liaoxuefeng.com/books/java/jdbc/basic/index.html)、[Oracle JDBC 入门](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html)。
