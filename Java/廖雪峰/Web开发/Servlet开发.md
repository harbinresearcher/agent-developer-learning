# Servlet开发

> **重要程度：中**｜**手练：建议**
> **重点**：参数、响应与编码。

## 核心概念

Servlet 从请求对象读取参数、头部等信息，通过响应对象设置状态码、头部和正文。

## 工作中注意

检查输入合法性与编码；写入响应后机遇已提交等状态可能限制修改头部。

## 手练

读取一个查询参数并返回结果，分别测试缺失值和非法值。

来源：[教程：Servlet开发](https://liaoxuefeng.com/books/java/web/servlet-dev/index.html)。

## 关联知识与实践

实现一个简单请求处理器，读取查询参数并返回结果；比较重定向和转发，再用 Filter 记录请求耗时，注意不记录敏感参数。

来源：[教程：Web开发](https://liaoxuefeng.com/books/java/web/index.html)；[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
