# HTTP编程

> **重要程度：高**｜**手练：需要**
> **重点**：请求、响应、超时和状态码。

## 核心概念

HTTP 客户端向服务器发送请求并读取响应。Java 11 的标准库提供 HttpClient，第三方库也可能由项目采用。

## 工作中注意

设定超时，校验状态码和数据，限制响应大小；重试前判断幂等性及是否可能重复写入。

## 手练

调用本地测试接口，分别验证成功、404 和超时处理。

来源：[教程：HTTP编程](https://liaoxuefeng.com/books/java/network/http/index.html)。

## 关联知识与实践

HTTP 请求包含方法、目标、头部和可选消息体；响应包含状态码、头部和消息体。GET 通常用于读取，POST 常用于提交处理；PUT、DELETE 等方法有各自语义。HTTP 状态成功不代表业务操作一定成功，还要检查响应内容。

Java 11 起标准库提供 `java.net.http.HttpClient`。工作中要设置合适的连接和请求超时，检查状态码，限制响应大小，正确关闭资源。重试前判断操作是否幂等，避免重复扣款、重复写入；必要时用业务幂等键。

调用本地测试接口，分别处理 200、404 和超时；对 POST 请求设计幂等键，并解释网络超时后为什么不能盲目重复提交。

来源：[教程：网络编程](https://liaoxuefeng.com/books/java/network/index.html)；[Java HttpClient](https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/java/net/http/HttpClient.html)。
