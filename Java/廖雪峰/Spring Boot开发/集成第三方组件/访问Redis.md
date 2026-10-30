# 访问Redis

> **重要程度：高**｜**手练：建议**
> **重点**：缓存、连接和失效。

## 核心概念

Redis 常用于缓存、限流或其他数据结构需求；Boot 可通过兼容客户端和配置访问。

## 工作中注意

缓存不是持久数据库的自动替代；考虑超时、失效、数据一致性和连接池。

## 手练

为测试查询加缓存，验证缓存未命中、更新和失效后的结果。

来源：[教程：访问Redis](https://liaoxuefeng.com/books/java/springboot/integration/redis/index.html)。
