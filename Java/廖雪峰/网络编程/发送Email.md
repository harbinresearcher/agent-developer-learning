# 发送 Email

> **重要程度：中**｜**手练：建议**
> **重点**：SMTP、MIME 构造、TLS、认证和发送结果的边界。

## 先理解链路

应用用 Jakarta Mail 构造 MIME 邮件，再通过 SMTP 服务提交。邮件对象构造成功不代表发送成功；SMTP 服务接受请求也不等于收件人一定收到或阅读。服务商的主机、端口、TLS 和认证方式必须按其文档配置。
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

## 离线练习：构造但不发送

保存为 `src/main/java/MailComposeDemo.java`：

```java
import jakarta.mail.Message;
import jakarta.mail.Session;
import jakarta.mail.internet.InternetAddress;
import jakarta.mail.internet.MimeMessage;
import java.util.Properties;
import java.util.logging.Logger;
public class MailComposeDemo {
    private static final Logger logger = Logger.getLogger(MailComposeDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：构造本地邮件对象
         * ========================================================================
         * 数据源：演示地址和固定文本
         * 操作：
         * 1) 创建 MIME 邮件并设置字段
         * 2) 检查邮件内容，不连接服务器
         */
        logger.info("开始构造演示邮件");
        // 1.1 在内存创建邮件对象
        Session session = Session.getInstance(new Properties());
        MimeMessage message = new MimeMessage(session);
        message.setFrom(new InternetAddress("sender@example.invalid"));
        message.setRecipients(Message.RecipientType.TO, InternetAddress.parse("nobody@example.invalid"));
        message.setSubject("学习示例", "UTF-8");
        message.setText("你好", "UTF-8");
        message.saveChanges();
        // 1.2 查看对象字段；此处不发送邮件
        System.out.println("subject=" + message.getSubject());
        System.out.println("recipients=" + message.getAllRecipients().length);
        System.out.println("text=" + message.isMimeType("text/plain"));
        logger.info("演示邮件构造完成");
    }
}
```

运行 `java -cp "target/classes;$notesDeps" MailComposeDemo`。标准输出：

```text
subject=学习示例
recipients=1
text=true
```

示例只在内存中创建邮件，不调用 Transport、不连接 SMTP，也不会发出邮件。`saveChanges()` 完成必要的 MIME 头部更新；读取主题、收件人数量和媒体类型用于观察对象。

## 实际 SMTP 示例：会产生外部发送

以下示例需要支持 587 端口 STARTTLS 和用户名/令牌认证的服务商。它从运行环境读取 `MAIL_HOST`、`MAIL_USER`、`MAIL_TOKEN` 和 `MAIL_TO`；`MAIL_USER` 在示例中也作为发件地址。真实运行会发送到 `MAIL_TO`，只应在有权限的测试邮箱和收件地址上操作。示例已编译，但没有连接真实 SMTP 或验证送达。

保存为 `src/main/java/SmtpSendDemo.java`，重新执行 `mvn package`，配置好环境后可用 `java -cp "target/classes;$notesDeps" SmtpSendDemo` 运行：

```java
import jakarta.mail.Authenticator;
import jakarta.mail.Message;
import jakarta.mail.PasswordAuthentication;
import jakarta.mail.Session;
import jakarta.mail.Transport;
import jakarta.mail.internet.InternetAddress;
import jakarta.mail.internet.MimeMessage;
import java.util.Objects;
import java.util.Properties;
import java.util.logging.Logger;
public class SmtpSendDemo {
    private static final Logger logger = Logger.getLogger(SmtpSendDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：通过 SMTP 发送一封邮件
         * ========================================================================
         * 数据源：外部配置的服务、账户、令牌和收件地址
         * 操作：
         * 1) 配置 STARTTLS 和认证并创建邮件
         * 2) 发送邮件，不记录凭据
         */
        logger.info("开始准备 SMTP 邮件");
        // 1.1 从运行环境读取配置
        String host = Objects.requireNonNull(System.getenv("MAIL_HOST"), "缺少 MAIL_HOST");
        String user = Objects.requireNonNull(System.getenv("MAIL_USER"), "缺少 MAIL_USER");
        String token = Objects.requireNonNull(System.getenv("MAIL_TOKEN"), "缺少 MAIL_TOKEN");
        String recipient = Objects.requireNonNull(System.getenv("MAIL_TO"), "缺少 MAIL_TO");
        Properties props = new Properties();
        props.setProperty("mail.smtp.host", host);
        props.setProperty("mail.smtp.port", "587");
        props.setProperty("mail.smtp.auth", "true");
        props.setProperty("mail.smtp.starttls.enable", "true");
        props.setProperty("mail.smtp.starttls.required", "true");
        props.setProperty("mail.smtp.ssl.checkserveridentity", "true");
        props.setProperty("mail.smtp.connectiontimeout", "5000");
        props.setProperty("mail.smtp.timeout", "5000");
        props.setProperty("mail.smtp.writetimeout", "5000");
        Session session = Session.getInstance(props, new Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(user, token);
            }
        });
        // 1.2 构造并发送邮件；此操作会联系服务商
        MimeMessage message = new MimeMessage(session);
        message.setFrom(new InternetAddress(user));
        message.setRecipients(Message.RecipientType.TO, InternetAddress.parse(recipient, true));
        message.setSubject("学习示例", "UTF-8");
        message.setText("这是一封测试邮件。", "UTF-8");
        Transport.send(message);
        logger.info("SMTP 发送调用完成");
    }
}
```

示例启用并要求 STARTTLS，设置连接和读写超时，并要求校验服务端身份。它只检查环境变量是否为 null；生产代码还需校验空值、发件人和收件人、内容大小及配置。465 端口的隐式 TLS、OAuth2 或不同认证方式不能照搬本例，应按服务商文档和客户端版本配置。

## 工作中怎么用

凭据来自受控密钥管理，不能写进源码、日志或版本库；开发环境的环境变量只是演示，不等同于完整的密钥管理。协议调试输出可能泄露敏感数据，应谨慎配置。邮件地址、标题和正文可能含隐私，不应随意记录。

异常、超时、部分收件人失败都要按服务商和业务处理。超时可能发生在服务已接收之后，盲目重试可能造成重复邮件；需要设计任务状态、幂等策略及可观测性。附件需要限制大小、类型和文件名，并按 MIME 规则构造。

## 手练与答案

1. 只运行离线构造示例：不会发邮件；它只能验证 MIME 对象构造。
2. SMTP 连接或认证失败：发送调用会抛异常；不能把“进入发送方法”当作投递成功。
3. `MAIL_USER` 不是合法发件地址时，需要按服务商规则分开发件地址与登录账号；本例不能直接满足这种配置。

参考：[廖雪峰：发送Email](https://liaoxuefeng.com/books/java/network/send-email/index.html)；[Eclipse Angus Mail](https://eclipse-ee4j.github.io/angus-mail/)。
