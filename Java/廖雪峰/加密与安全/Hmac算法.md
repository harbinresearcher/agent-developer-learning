# Hmac算法

> **重要程度：高**｜**手练：需要**
> **重点**：共享密钥、消息认证、验证与重放边界。

## 解决什么问题

HMAC 使用共享密钥和消息计算认证码。验证方用同一密钥对收到的**约定字节**重新计算，并比较认证码，从而检查消息是否被更改、是否由持有共享密钥的一方生成。它不加密消息，也不会自动防重放。

## 完整例子

保存为 `HmacDemo.java`，编译运行；日志写到标准错误。`test-key` 是公开教学数据，绝不能作为真实密钥。

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.Base64;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.util.logging.Logger;
public class HmacDemo {
    private static final Logger logger = Logger.getLogger(HmacDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：生成并验证示例认证码
         * ========================================================================
         * 数据源：公开的教学测试密钥和非敏感消息
         * 操作：
         * 1) 使用 HmacSHA256 计算消息认证码
         * 2) 用同一输入重新计算并比较
         */
        logger.info("开始验证示例消息");
        // 1.1 使用教学密钥计算消息认证码
        byte[] key = "test-key".getBytes(StandardCharsets.UTF_8);
        byte[] message = "amount=100".getBytes(StandardCharsets.UTF_8);
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(key, "HmacSHA256"));
        byte[] tag = mac.doFinal(message);
        // 1.2 重新计算并比较原始字节
        byte[] expected = mac.doFinal(message);
        System.out.println("tag=" + Base64.getEncoder().encodeToString(tag));
        System.out.println("same=" + MessageDigest.isEqual(tag, expected));
        logger.info("示例消息验证完成");
    }
}
```

标准输出：

```text
tag=emO/qAubEFH41iUaJMwZfWzjuCLlvmj1keyF2OArgfw=
same=true
```

## 执行过程

示例将固定消息按 UTF-8 编码，以教学密钥初始化 `HmacSHA256`，计算认证码，再对相同字节重新计算并比较。两次结果相同，所以输出 true；Base64 只便于展示认证码，不提供保密性。示例中的第二次计算与第一次在同一进程完成，只验证 API 用法，不构成真实的端到端认证。

## 工作中必须明确的协议

认证双方必须对完全相同的字节计算：包括字符编码、字段顺序、分隔方式、时间戳和请求体等。不要自行拼接含歧义的字符串设计签名协议，应使用成熟的标准协议或服务方规定。校验时使用适当的比较 API，避免普通字符串比较造成不必要的时间侧信道；示例使用 `MessageDigest.isEqual` 比较字节，但整体协议仍需要安全设计。

同一条有效消息可能被重复发送；防重放需要结合随机数或序列号、时效窗口与服务端去重等机制。密钥应由安全随机机制生成并通过密钥管理系统存储、轮换；不要写进源码、日志或仓库。任何持有同一共享密钥的一方都可能生成认证码，因此 HMAC 不提供数字签名那种公私钥分离属性。

## 手练与答案

将第二次计算的消息改成不同字节，比较结果应为 false。把认证码 Base64 编码不会隐藏消息；同一消息和认证码若被原样重放，本示例没有防重放措施。

参考：[Oracle JCA 指南](https://docs.oracle.com/en/java/javase/17/security/java-cryptography-architecture-jca-reference-guide.html)。

课程原文：[廖雪峰：Hmac算法](https://liaoxuefeng.com/books/java/security/hmac/index.html)。
