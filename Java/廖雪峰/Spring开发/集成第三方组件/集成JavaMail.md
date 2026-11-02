# 集成JavaMail

> **重要程度：中**｜**手练：需要**
> **重点**：JavaMailSender、MIME、编码、连接配置和发送失败。

## 先理解邮件发送链

应用构造邮件内容，通过 SMTP 客户端将邮件提交给邮件服务器；服务器接受请求不代表收件人一定收到。Spring 的 `JavaMailSender` 提供发送抽象，`MimeMessageHelper` 方便设置主题、正文和收件人。现代版本使用 Jakarta Mail，旧教程中的 `javax.mail` 与 `jakarta.mail` 不应混用。

下面的例子只在内存中构造并读取 MIME 消息，不连接服务器，也不发送邮件。地址使用保留的演示域名。

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
    <groupId>learn.example</groupId><artifactId>spring-mail-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-mail</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

保存 `src/main/java/learn/mail/MailDemo.java`：

```java
package learn.mail;
import jakarta.mail.Message;
import jakarta.mail.internet.MimeMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;
public class MailDemo {
    private static final Logger logger = LoggerFactory.getLogger(MailDemo.class);
    public static void main(String[] args) throws Exception {
        /* ========================================================================
         * 步骤1：构造并检查邮件
         * 数据源：演示用地址、主题和正文
         * 操作：1) 创建 MIME 消息 2) 检查消息字段
         * ======================================================================== */
        logger.info("开始构造邮件");
        // 1.1 离线构造消息，不连接 SMTP 服务
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        MimeMessage message = sender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, false, "UTF-8");
        helper.setFrom("sender@example.test");
        helper.setTo("reader@example.test");
        helper.setSubject("测试通知");
        helper.setText("订单已创建", false);
        message.saveChanges();
        // 1.2 检查主题与收件人，不发送消息
        System.out.println("主题=" + message.getSubject());
        System.out.println("收件人=" + message.getRecipients(Message.RecipientType.TO)[0]);
        logger.info("邮件构造完成");
    }
}
```

先用 `java -version` 和 `mvn -v` 确认使用 JDK 17 或以上。在工程目录运行：

```powershell
mvn package
java -jar target/spring-mail-demo-1.0.jar
```

日志中应看到：

```text
主题=测试通知
收件人=reader@example.test
```

`MimeMessageHelper` 使用 UTF-8 设置中文主题与纯文本正文；`saveChanges()` 更新消息头后可读取字段。例子没有调用 `send`，因此不能证明任何 SMTP 配置或投递结果。

## 真实项目需要补什么

Spring Boot 在满足邮件库、主机配置等条件时可自动配置 `JavaMailSender`。真实发送需按邮件服务商要求配置主机、端口、认证、TLS 和连接、读取、写入超时；密码或授权码应由受控密钥渠道提供，不写入源码或日志。

发送可能超时或失败，且超时后结果可能不确定。异步发送和重试要考虑重复通知、幂等、失败记录与告警；如果与数据库事务关联，还要明确事务提交和消息发送的顺序。HTML 邮件对不可信内容按 HTML 上下文转义，附件要限制大小与来源。

## 动手与自查

1. 修改主题和收件人，运行并检查读取结果。
2. 查项目所用邮件服务商文档，列出 TLS、认证和超时所需配置，先在测试环境验证。
3. 为什么 SMTP 提交成功或客户端超时都不能单独证明收件人已读到邮件？

来源：[教程：集成JavaMail](https://liaoxuefeng.com/books/java/spring/integration/javamail/index.html)、[Spring Boot 邮件](https://docs.spring.io/spring-boot/reference/io/email.html)、[Spring 邮件支持](https://docs.spring.io/spring-framework/reference/integration/email.html)。
