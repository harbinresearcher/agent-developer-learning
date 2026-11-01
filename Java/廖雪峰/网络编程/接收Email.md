# 接收 Email

> **重要程度：中**｜**手练：建议**
> **重点**：IMAP/POP3、只读访问、MIME 解析、凭据与资源关闭。

## 先理解链路

IMAP 和 POP3 是读取邮箱的协议，语义和服务商支持情况不同。IMAP 通常以服务器上的文件夹和消息为中心；读取后还需要按 MIME 类型解析纯文本、HTML、多部分正文和附件。邮件内容来自外部，不应默认可信。
## 依赖与运行

以下完整 `pom.xml` 使用 Jakarta Mail 的 Angus 实现；编译目标为 Java 11。旧项目可能使用 `javax.mail`，不能不核对版本就和 `jakarta.mail` 包名混用。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>example.notes</groupId><artifactId>mail-demo</artifactId><version>1.0.0</version>
  <properties><maven.compiler.release>11</maven.compiler.release><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding></properties>
  <dependencies>
    <dependency><groupId>org.eclipse.angus</groupId><artifactId>angus-mail</artifactId><version>2.0.5</version></dependency>
  </dependencies>
  <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

将本页 Java 文件放进项目的 `src/main/java`，在 pom 所在目录的 Windows PowerShell 运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$notesDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
```

随后按下方命令启动目标类。首次运行可能需要下载依赖。本次用 Maven 3.9.16、JDK 17 和 Java 11 编译目标构建，并用 Java 11 运行本地 MIME 示例；没有连接真实邮箱或验证投递。示例没有配置自动测试，构建成功不等于真实邮件功能通过集成测试。

## 离线练习：解析一封内存邮件

保存为 `src/main/java/ReadMimeDemo.java`：

```java
import jakarta.mail.Session;
import jakarta.mail.internet.MimeMessage;
import java.io.ByteArrayInputStream;
import java.nio.charset.StandardCharsets;
import java.util.Properties;
import java.util.logging.Logger;
public class ReadMimeDemo {
    private static final Logger logger = Logger.getLogger(ReadMimeDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：解析本地演示邮件
         * ========================================================================
         * 数据源：程序中固定的 MIME 邮件文本
         * 操作：
         * 1) 创建邮件对象并解析头部
         * 2) 检查文本正文
         */
        logger.info("开始解析演示邮件");
        // 1.1 准备受控的 MIME 字节，不连接邮箱
        String raw = "From: sender@example.invalid\r\n"
                + "To: nobody@example.invalid\r\n"
                + "Subject: example\r\n"
                + "Content-Type: text/plain; charset=UTF-8\r\n\r\n"
                + "hello";
        Session session = Session.getInstance(new Properties());
        MimeMessage message = new MimeMessage(session,
                new ByteArrayInputStream(raw.getBytes(StandardCharsets.UTF_8)));
        // 1.2 在已知纯文本示例中读取字段
        System.out.println("subject=" + message.getSubject());
        System.out.println("content=" + message.getContent());
        logger.info("演示邮件解析完成");
    }
}
```

运行 `java -cp "target/classes;$notesDeps" ReadMimeDemo`，标准输出：

```text
subject=example
content=hello
```

这个例子只解析程序内的固定 MIME 文本，不登录邮箱。它已运行验证；真实邮件可能是 multipart 或附件，`getContent()` 不总是 String，不能在生产代码中无条件强制转换。

## 实际 IMAPS 示例：只读查询数量

以下示例会连接 `MAIL_HOST` 指定的 IMAPS 服务，使用 `MAIL_USER` 和 `MAIL_TOKEN` 登录并只读打开 INBOX；实际输出取决于邮箱内容。只应访问有权限的测试邮箱。示例已编译，但没有连接真实邮箱或验证服务商认证。部分服务商要求 OAuth2 等机制，此示例的用户名/令牌方式不适用。

保存为 `src/main/java/ImapReadDemo.java`，重新执行 `mvn package`，配置环境后可运行 `java -cp "target/classes;$notesDeps" ImapReadDemo`：

```java
import jakarta.mail.Folder;
import jakarta.mail.Session;
import jakarta.mail.Store;
import java.util.Objects;
import java.util.Properties;
import java.util.logging.Logger;
public class ImapReadDemo {
    private static final Logger logger = Logger.getLogger(ImapReadDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：只读查询收件箱数量
         * ========================================================================
         * 数据源：外部配置的 IMAPS 服务和账户
         * 操作：
         * 1) 建立 TLS 连接并只读打开收件箱
         * 2) 查询数量并关闭资源
         */
        logger.info("开始查询收件箱");
        // 1.1 读取配置并设置连接和读取超时
        String host = Objects.requireNonNull(System.getenv("MAIL_HOST"), "缺少 MAIL_HOST");
        String user = Objects.requireNonNull(System.getenv("MAIL_USER"), "缺少 MAIL_USER");
        String token = Objects.requireNonNull(System.getenv("MAIL_TOKEN"), "缺少 MAIL_TOKEN");
        Properties props = new Properties();
        props.setProperty("mail.imaps.ssl.checkserveridentity", "true");
        props.setProperty("mail.imaps.connectiontimeout", "5000");
        props.setProperty("mail.imaps.timeout", "5000");
        Session session = Session.getInstance(props);
        // 1.2 建立连接并只读访问收件箱
        try (Store store = session.getStore("imaps")) {
            store.connect(host, user, token);
            Folder inbox = store.getFolder("INBOX");
            inbox.open(Folder.READ_ONLY);
            try {
                System.out.println("count=" + inbox.getMessageCount());
            } finally {
                inbox.close(false);
            }
        }
        logger.info("收件箱查询完成");
    }
}
```

`imaps` 使用 TLS；示例启用服务端身份检查，设置连接和读取超时，用 `Folder.READ_ONLY` 打开文件夹，并以 `close(false)` 关闭，避免请求清除已删除标记的邮件。具体端口、认证和服务器行为以服务商及客户端文档为准；只读不等于不产生服务器访问记录。

## 工作中怎么用

按服务商授权范围访问，凭据通过受控配置提供，不写日志或仓库。邮件标题、正文和附件可能包含隐私或恶意内容；限制大小，验证类型，安全处理文件名，HTML 不应直接作为可信内容展示。大邮箱需要分页或增量读取，避免一次加载所有邮件。

处理断连、超时、重复处理与状态同步；若需要改变已读或删除状态，应明确权限和业务语义。POP3 的文件夹、状态和同步能力与 IMAP 不同，不应直接套用本例。

## 手练与答案

1. 离线示例中把 Subject 改为 `note`：输出 subject=note，不会连接邮箱。
2. 主机或凭据错误时实际连接可能失败；离线 MIME 测试通过不能证明真实邮箱可访问。
3. 收到 multipart 邮件时 `getContent()` 可能返回 Multipart，需要按结构递归处理，而不是假设一定是字符串。

参考：[廖雪峰：接收Email](https://liaoxuefeng.com/books/java/network/receive-email/index.html)；[Eclipse Angus Mail](https://eclipse-ee4j.github.io/angus-mail/)。
