# SMTP 发送邮件

> **重要程度：低**｜**手练：按需**
> 重点：邮件构造、TLS、凭据保护。

## 发送流程

`email.message.EmailMessage` 用于构造邮件，`smtplib` 用于连接 SMTP 服务并发送。邮件头中的 From、To、Subject 是消息内容的一部分；SMTP 信封发件人与收件人用于投递，二者含义不同。

服务商通常提供隐式 TLS 或 STARTTLS：前者常用 465 端口，后者常用 587 端口；具体地址、端口及认证方式以服务商文档为准。用默认安全 SSL 上下文验证证书，设置连接超时，并在结束时关闭连接。

## 工作中注意

- 凭据可能是授权码、应用密码或其他认证方式，按服务商要求配置；不要提交源码或写入日志。
- `sendmail` 可接收单个收件人地址字符串，也可接收地址序列；多人发送时注意隐私。现代 `EmailMessage` 可按邮件策略处理头部编码。
- 邮件服务器接受投递不等于最终进入收件箱。处理部分收件人被拒、网络错误和重复发送；日志不要记录邮件正文或凭据。
- 不要在生产环境开启 SMTP 协议调试输出，避免泄露敏感信息。

手练：用本地测试 SMTP 服务发送一封纯文本邮件，检查信封收件人、消息头和超时处理。真实发送前使用自己的测试地址。

参考：[Python email 示例](https://docs.python.org/3/library/email.examples.html)；[Python smtplib](https://docs.python.org/3/library/smtplib.html)。
