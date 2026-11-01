# Servlet入门

> **重要程度：中**｜**手练：建议**
> **重点**：容器、映射和生命周期。

## 核心概念

Servlet 容器接收请求并调用映射到对应路径的应用组件；Servlet 对象通常由容器创建和管理。

## 工作中注意

同一 Servlet 可能处理并发请求，不应把请求专属可变数据存入共享实例字段。

## 手练

在测试容器中建立最小 Servlet，验证请求路径映射和响应状态。

来源：[教程：Servlet入门](https://liaoxuefeng.com/books/java/web/servlet-basic/index.html)。

## 关联知识与实践

浏览器或客户端发送 HTTP 请求，Web 服务器接收请求并交给 Servlet 容器；容器按照映射找到 Servlet 或 Filter，再调用应用代码。响应包含状态码、头部和正文。Servlet 对象通常由容器管理，请求可并发到达；不要把每个请求的数据存放在共享的可变实例字段中。

Java EE 已演进为 Jakarta EE。现代版本常使用 `jakarta.*` 命名空间；旧教程或项目可能仍使用 `javax.*`，不能不核对框架版本就混用。

来源：[教程：Web开发](https://liaoxuefeng.com/books/java/web/index.html)；[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
