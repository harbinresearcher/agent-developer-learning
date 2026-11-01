# Hmac算法

> **重要程度：高**｜**手练：建议**
> **重点**：共享密钥、消息认证和比较。

## 核心概念

HMAC 使用密钥和消息生成认证码，可验证消息完整性并确认发送方持有共享密钥；它不提供消息保密性。

## 工作中注意

密钥需要保密；验证要采用适当的安全比较方式。HMAC 自身不阻止重放，也不适合单次用于密码存储。

## 手练

生成测试消息的 HMAC，修改消息后验证失败；说明时间戳与重放防护的关系。

来源：[教程：Hmac算法](https://liaoxuefeng.com/books/java/security/hmac/index.html)。

## 关联知识与实践

Base64 是编码，不提供保密性。SHA-256 等摘要用于完整性校验；若攻击者能同时替换内容和摘要，普通摘要不能证明来源。HMAC 用共享密钥校验消息完整性和真实性；它不加密内容，也不能防止重放，协议可能需要时间戳或随机数。

来源：[教程：加密与安全](https://liaoxuefeng.com/books/java/security/index.html)；[OWASP 密码存储](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)。
