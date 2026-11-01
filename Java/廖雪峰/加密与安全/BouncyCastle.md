# Bouncy Castle

> **重要程度：中**｜**手练：按需**
> **重点**：JCA Provider、依赖版本、显式选择与安全边界。

## 解决什么问题

Java Cryptography Architecture（JCA）将算法 API 与具体提供者分开。Bouncy Castle 是提供密码学实现及相关工具的第三方项目；某些算法、格式或协议在项目中需要它。JDK 已提供的算法并不一定需要额外引入第三方 provider。

## 完整例子

保存为 `ProviderDemo.java`。将 [bcprov-jdk18on-1.86.jar](https://repo.maven.apache.org/maven2/org/bouncycastle/bcprov-jdk18on/1.86/bcprov-jdk18on-1.86.jar) 放在同一目录。以下命令适用于 Windows PowerShell；其他系统的类路径分隔符不同。

```text
javac -encoding UTF-8 -cp bcprov-jdk18on-1.86.jar ProviderDemo.java
java -cp ".;bcprov-jdk18on-1.86.jar" ProviderDemo
```

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.Provider;
import org.bouncycastle.jce.provider.BouncyCastleProvider;
import java.util.logging.Logger;
public class ProviderDemo {
    private static final Logger logger = Logger.getLogger(ProviderDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：使用指定的加密提供者
         * ========================================================================
         * 数据源：公开的测试文本与 Bouncy Castle provider
         * 操作：
         * 1) 使用指定 provider 创建摘要实例
         * 2) 计算摘要并检查长度
         */
        logger.info("开始计算测试摘要");
        // 1.1 创建 provider 并明确使用它
        Provider provider = new BouncyCastleProvider();
        MessageDigest digest = MessageDigest.getInstance("SHA-256", provider);
        // 1.2 计算非敏感数据的摘要长度
        byte[] result = digest.digest("abc".getBytes(StandardCharsets.UTF_8));
        System.out.println("provider=" + digest.getProvider().getName());
        System.out.println("bytes=" + result.length);
        logger.info("测试摘要计算完成");
    }
}
```

标准输出为 `provider=BC` 和 `bytes=32`；日志写到标准错误。示例已用 Java 11 和 Bouncy Castle 1.86 编译运行。版本只是本次验证配置，不代表永远是最新或适合所有项目。

## 执行过程

`new BouncyCastleProvider()` 创建 provider 对象；`MessageDigest.getInstance("SHA-256", provider)` 为这个调用显式指定实现。计算 UTF-8 的 `abc` 后得到 32 字节摘要，即 256 位。本例没有把 provider 全局注册进 JVM；如果使用 `Security.addProvider`，它会影响当前 JVM 的 provider 列表和可能的查找顺序，应由应用统一管理。

## 依赖与兼容

Maven 的 `<dependencies>` 中可加入以下片段；这不是完整 pom：

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk18on</artifactId>
    <version>1.86</version>
</dependency>
```

`jdk18on` 指 Java 1.8 及更高版本的发布线，不是“只能用于 Java 18”；具体兼容性以官方该版本文档为准。其他功能可能需要不同 Bouncy Castle 工件；不要随意混用版本。某个 provider 存在不等于所有算法或配置都安全，需核对项目策略和官方安全公告。

## 工作中怎么用

优先使用经过审查的高层库和标准协议；新增 provider 前确认算法需求、供应链来源、许可证、升级策略和是否已有等效实现。不要把 provider 选择当成密钥管理或身份认证的替代品。

## 手练与答案

本例 `digest.getProvider().getName()` 返回 `BC`，摘要长度是 32 字节。移除 jar 后编译器无法找到 `BouncyCastleProvider`；SHA-256 本身仍可能由 JDK 内置 provider 提供，但那已不是本例指定的实现。

参考：[廖雪峰：BouncyCastle](https://liaoxuefeng.com/books/java/security/bouncy-castle/index.html)；[Bouncy Castle 下载](https://www.bouncycastle.org/download/bouncy-castle-java/)；[Oracle JCA 指南](https://docs.oracle.com/en/java/javase/17/security/java-cryptography-architecture-jca-reference-guide.html)。
