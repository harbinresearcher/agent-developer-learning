# 集成Filter

> **重要程度：中**｜**手练：建议**
> **重点**：Servlet 过滤链与顺序。

## 核心概念

Filter 在 Servlet 层处理请求链，可用于编码、认证上下文或通用日志等。

## 工作中注意

过滤器顺序影响安全与行为；避免记录敏感头或无界缓存请求体。

## 手练

添加测试过滤器，验证请求进入、响应返回和异常路径的顺序。

来源：[教程：集成Filter](https://liaoxuefeng.com/books/java/spring/web/filter/index.html)。
