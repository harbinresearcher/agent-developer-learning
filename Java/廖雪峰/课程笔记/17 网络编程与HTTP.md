# 网络编程与 HTTP

> **重要程度：高**｜**手练：需要**
> **重点**：TCP 字节流、HTTP 请求响应、超时、状态码和重试边界。

TCP 面向连接并提供有序字节流，但不保留应用消息边界；一次 `read` 不保证读到完整消息，需要按长度或分隔符等协议分帧。UDP 面向数据报，不保证交付、顺序或去重。低层 Socket 适合学习协议；调用普通 HTTP API 时优先使用成熟 HTTP 客户端。

HTTP 请求包含方法、目标、头部和可选消息体；响应包含状态码、头部和消息体。GET 通常用于读取，POST 常用于提交处理；PUT、DELETE 等方法有各自语义。HTTP 状态成功不代表业务操作一定成功，还要检查响应内容。

Java 11 起标准库提供 `java.net.http.HttpClient`。工作中要设置合适的连接和请求超时，检查状态码，限制响应大小，正确关闭资源。重试前判断操作是否幂等，避免重复扣款、重复写入；必要时用业务幂等键。

邮件发送涉及 SMTP、服务商认证和 TLS，按服务商接口配置。旧 RMI 技术了解用途即可，不作为新项目首选通信方式。

## 手练

调用本地测试接口，分别处理 200、404 和超时；对 POST 请求设计幂等键，并解释网络超时后为什么不能盲目重复提交。

来源：[教程：网络编程](https://liaoxuefeng.com/books/java/network/index.html)；[Java HttpClient](https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/java/net/http/HttpClient.html)。
