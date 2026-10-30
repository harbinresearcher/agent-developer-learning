# 使用Session和Cookie

> **重要程度：高**｜**手练：建议**
> **重点**：客户端 Cookie 与服务端会话。

## 核心概念

Cookie 由客户端保存并随合适请求发送；Session 是服务端管理的一种会话状态，通常用标识关联请求。

## 工作中注意

按部署需求设置 Secure、HttpOnly、SameSite 和过期策略；防范会话固定、跨站请求伪造等风险。

## 手练

在测试应用中创建会话并检查 Cookie 属性，再验证退出登录后的会话状态。

来源：[教程：使用Session和Cookie](https://liaoxuefeng.com/books/java/web/servlet-adv/session-cookie/index.html)。
