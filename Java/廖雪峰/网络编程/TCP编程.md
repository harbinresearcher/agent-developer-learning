# TCP编程

> **重要程度：中**｜**手练：建议**
> **重点**：连接、分帧与超时。

## 核心概念

TCP 是面向连接的字节流，读写可能分多次完成；应用层需约定消息边界，例如长度前缀。

## 工作中注意

一次 read 不保证读完一条消息；要限制大小、设置超时并可靠关闭连接。

## 手练

在本地建立测试客户端和服务器，发送分段数据，验证接收端按协议正确重组。

来源：[教程：TCP编程](https://liaoxuefeng.com/books/java/network/tcp/index.html)。

## 关联知识与实践

TCP 面向连接并提供有序字节流，但不保留应用消息边界；一次 `read` 不保证读到完整消息，需要按长度或分隔符等协议分帧。UDP 面向数据报，不保证交付、顺序或去重。低层 Socket 适合学习协议；调用普通 HTTP API 时优先使用成熟 HTTP 客户端。

来源：[教程：网络编程](https://liaoxuefeng.com/books/java/network/index.html)；[Java HttpClient](https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/java/net/http/HttpClient.html)。
