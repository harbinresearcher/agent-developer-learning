# 集成JMX

> **重要程度：中**｜**手练：需要**
> **重点**：MBean、对象名、属性与操作、本地和远程管理边界。

## JMX 做什么

JMX 是 Java 的管理扩展机制。受管理对象可以作为 MBean 注册到 `MBeanServer`，暴露属性和操作，供管理工具访问。Spring 可以根据注解将 Bean 导出为 MBean。

例子使用 `@ManagedResource` 指定对象名，用 `@ManagedAttribute` 暴露只读计数，用 `@ManagedOperation` 暴露递增操作。程序通过同一 JVM 的平台 MBeanServer 读取和调用，没有开启远程管理端口。

## 完整例子

使用 JDK 17、Maven 和 Spring Boot 4.1.2。新建目录，保存 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>spring-jmx-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/jmx/JmxDemo.java`：

```java
package learn.jmx;
import java.lang.management.ManagementFactory;
import javax.management.MBeanServer;
import javax.management.ObjectName;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.EnableMBeanExport;
import org.springframework.jmx.export.annotation.ManagedAttribute;
import org.springframework.jmx.export.annotation.ManagedOperation;
import org.springframework.jmx.export.annotation.ManagedResource;
@SpringBootApplication
@EnableMBeanExport
public class JmxDemo {
    private static final Logger logger = LoggerFactory.getLogger(JmxDemo.class);
    @ManagedResource(objectName = "learn:type=Counter")
    public static class Counter {
        private int count;
        @ManagedAttribute public int getCount() { return count; }
        @ManagedOperation public int increment() { return ++count; }
    }
    @Bean Counter counter() { return new Counter(); }
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：访问本地管理对象
         * 数据源：本 JVM 的 MBeanServer
         * 操作：1) 读取属性 2) 调用操作并检查结果
         * ======================================================================== */
        logger.info("开始 JMX 示例");
        try (ConfigurableApplicationContext context = SpringApplication.run(JmxDemo.class, args)) {
            // 1.1 通过对象名读取当前属性
            MBeanServer server = ManagementFactory.getPlatformMBeanServer();
            ObjectName name = new ObjectName("learn:type=Counter");
            System.out.println("初始=" + server.getAttribute(name, "Count"));
            // 1.2 在同一 JVM 内调用管理操作
            System.out.println("递增=" + server.invoke(name, "increment", new Object[0], new String[0]));
        }
        logger.info("JMX 示例完成");
    }
}
```

先确认 `java -version` 和 `mvn -v` 使用 JDK 17 或以上，在工程目录运行：

```powershell
mvn package
java -jar target/spring-jmx-demo-1.0.jar
```

日志中应看到：

```text
初始=0
递增=1
```

`@EnableMBeanExport` 启用导出，`ObjectName` 标识 MBean。`getAttribute` 读取管理属性，`invoke` 调用操作；容器关闭后不应假设导出的对象仍然可用。计数器仅作单次调用演示，并未处理并发递增。

## 工作中注意

管理操作可能改变应用状态，应明确权限、审计和错误处理。远程 JMX 涉及认证、传输保护、网络访问和端口配置；不要将管理接口直接暴露给不可信网络。本例只验证了本地 MBeanServer，不能证明远程访问或安全配置正确。

监控指标与可修改操作的访问权限通常不同；生产项目应尽量暴露必要信息，避免泄漏敏感数据。多实例部署时，每个 JVM 的管理对象和状态也需要分别考虑。

## 动手与自查

1. 连续调用两次 `increment`，检查返回值。
2. 修改对象名，观察查询不到时的错误；再恢复一致的名称。
3. 为什么本机 MBean 操作成功不能证明远程管理接口已安全配置？

来源：[教程：集成JMX](https://liaoxuefeng.com/books/java/spring/integration/jmx/index.html)、[Spring JMX](https://docs.spring.io/spring-framework/reference/integration/jmx.html)。
