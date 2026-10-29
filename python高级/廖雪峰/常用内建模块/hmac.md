# hmac：消息认证

> **重要程度：中**｜**手练：建议**
> 重点：用 HMAC-SHA256 生成和验证消息认证码，理解密钥管理和比较方法。

## 用途

HMAC 用共享密钥和消息生成认证码。接收方用同一密钥重新计算并比较，可检测消息是否被篡改，并确认发送方持有密钥。它不加密消息，也不能证明是哪一个共享密钥的持有者发送的。

## 使用要点

- `hmac.new(key, message, digestmod='sha256').hexdigest()` 计算十六进制认证码。密钥使用字节，消息可使用字节或符合接口要求的 bytes-like 对象。
- 验证时用 `hmac.compare_digest(expected, actual)`，两者类型和格式需一致；它旨在减少内容相关的时序泄露，但类型和长度等信息仍可能泄露。
- 密钥由安全随机源生成，通过密钥管理服务或受控环境变量注入；不要硬编码、写入日志或提交仓库。密钥泄露后需要轮换。
- 对外部请求验签时，应严格遵守协议规定的原始消息、编码和签名格式；是否需要时间戳、随机数及重放防护由协议设计决定。

## 与密码存储的区别

HMAC 的密钥必须保密；密码盐通常与密码哈希一起保存。单次 HMAC 太快，不适合直接保存用户密码。密码存储见 [[hashlib]]。

## 自测

1. 普通 SHA-256 摘要和 HMAC 有什么区别？
2. 消息和认证码都能被修改时，为什么需要保密密钥？
3. 如何比较认证码？为什么单次 HMAC 不适合存密码？

参考：[Python hmac](https://docs.python.org/3/library/hmac.html)；[OWASP 密码存储指南](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)。
