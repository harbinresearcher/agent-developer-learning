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
