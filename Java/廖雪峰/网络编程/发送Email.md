# 发送Email

> **重要程度：低**｜**手练：按需**
> **重点**：SMTP、认证和 TLS。

## 核心概念

发送邮件通常通过 SMTP 服务提交，邮件头和 SMTP 信封地址各有作用。Java 项目可使用项目认可的邮件库。

## 工作中注意

按服务商配置 TLS 和认证方式；服务器接受不等于最终送达。不要在代码或日志中保存真实凭据。

## 手练

向本地测试邮件服务器提交一封邮件，检查标题、正文和信封收件人。

来源：[教程：发送Email](https://liaoxuefeng.com/books/java/network/send-email/index.html)。

## 关联知识与实践

邮件发送涉及 SMTP、服务商认证和 TLS，按服务商接口配置。旧 RMI 技术了解用途即可，不作为新项目首选通信方式。

来源：[教程：网络编程](https://liaoxuefeng.com/books/java/network/index.html)；[Java HttpClient](https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/java/net/http/HttpClient.html)。
