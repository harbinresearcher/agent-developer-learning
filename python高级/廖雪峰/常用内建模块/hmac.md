这页的内容可以顺着一个很自然的演进逻辑来理解：上一页的“加盐哈希”其实就是 HMAC 的手工版，而 HMAC 是这件事的标准化、更安全的官方做法。咱们按“为什么需要它 → 它到底是什么 → Python 怎么用 → 和加盐比好在哪 → 几个必须注意的坑”这条线走。

---

# hmac · 笔记

## 先看它从哪来：加盐哈希的“标准化版本”

上一页学哈希存密码，最后的结论是**每个用户一个随机盐**。当时的写法本质上是 `hash(message + salt)`，把盐拼接在消息后面再算哈希。

这个做法能用，但它是**你自己发明的**。自定义的拼接方式有没有漏洞、盐放前面还是后面更安全、换一种哈希算法这套拼接逻辑还成不成立——这些都是你自己在扛。HMAC 把“带密钥的哈希”这件事**标准化了**：它不是简单的 `hash(key + message)`，而是一套经过密码学分析的、固定的构造流程，**把 key 混入哈希计算过程中**，而不是粗暴地拼在开头或结尾。

所以一句话：**HMAC = 带密钥的哈希，而且这个“带”的方式是标准化的，不是你自己拍的。**

## Python 怎么用：`hmac.new(key, msg, digestmod)`

核心函数就一个 `hmac.new()`，参数顺序是 **`key` 在前，`message` 在后**，第三个参数指定底层用哪种哈希算法。

```python
import hmac

message = b'Hello, world!'                     # 待认证的消息，必须是 bytes
key = b'secret'                                # 密钥，必须是 bytes
h = hmac.new(key, message, digestmod='MD5')    # 用 MD5 作为底层哈希，key 混入计算
print(h.hexdigest())                           # 'fa4ee7d173f2d97ee79022d1a7355bcf'
```

**三个要点**：

第一，**`key` 和 `message` 都必须是 `bytes`**，字符串要先 `.encode('utf-8')`。

第二，**输出长度和底层哈希算法一致**。用 MD5 就是 32 位十六进制，用 SHA256 就是 64 位。HMAC 本身不改变哈希长度，它改变的是“计算时有没有混入 key”。

第三，**消息很长时可以分多次 `update()`**，和普通 hashlib 的用法一样。

## 和“加盐”的本质区别：为什么用它比手写强

原文有一句很关键的话：**HMAC 算法针对所有哈希算法都通用，无论是 MD5 还是 SHA-1。采用 HMAC 替代自定义的 salt 算法，可以使程序算法更标准化，也更安全**。

拆开说：

**标准化**：你手写 `hash(password + salt)`，换一个哈希算法，你得重新考虑拼接方式还对不对。HMAC 不管底层换成 MD5、SHA1 还是 SHA256，**构造流程不变**，你只需要换 `digestmod` 参数。

**更安全**：简单的 `hash(key + message)` 在某些哈希算法下存在**长度扩展攻击**的风险。HMAC 的构造（它实际上做了两次哈希，内外各一次）专门设计来抵抗这类攻击。你不需要自己发明拼接方案，标准已经替你考虑过了。

## 坑一：登录验证时，用 `compare_digest` 比较，别用 `==`

这是实际项目里最容易被忽略、但后果最直接的一个点。

```python
import hmac

def login(username, password):
    user = db[username]
    expected = user.password                         # 数据库里存的 HMAC 值
    actual = hmac_md5(user.key, password)            # 用本次输入算出的 HMAC
    return hmac.compare_digest(expected, actual)     # 用恒定时间比较，不要用 ==
```

**为什么不能用 `==`**：字符串的 `==` 比较是**短路**的，只要发现第一个不同的字符就立刻返回 `False`。这意味着**比较所花的时间，和“前面对上了多少个字符”相关**。攻击者可以通过反复请求、测量响应时间，逐位猜出正确的 HMAC 值——这叫**时序攻击**。

`hmac.compare_digest()` 的比较时间是**恒定的**，不管两个值在哪个位置不同，它都花一样长的时间，堵死了这个侧信道。

## 坑二：key 不能硬编码在代码里

原文练习里用了随机生成的 key，但实际项目里**密钥不应该出现在源码里**，更不能提交到 Git。密钥应该来自环境变量、密钥管理服务或配置文件（且配置文件不入库）。

```python
import os
key = os.environ['HMAC_SECRET'].encode('utf-8')    # 从环境变量读，不写死在代码里
```

## 坑三：HMAC 不是加密，key 泄露就全完了

和哈希一样，**HMAC 是不可逆的**，你不能从 HMAC 值反推出原始消息或 key。但它的安全性**完全建立在 key 保密的基础上**。key 一旦泄露，攻击者可以伪造任何消息的 HMAC，认证机制直接失效。

所以 HMAC 的正确用法是：**key 两边共享、严格保密、定期轮换**。它不是用来“保护消息内容”的（那是加密的事），而是用来“证明消息确实来自持有 key 的人、且没有被篡改”。

## 完整例子：用 HMAC 改写密码存储

原文的练习就是把上一页的盐替换成 HMAC，代码结构几乎不变，但标准化程度更高：

```python
import hmac, random

def hmac_md5(key, s):
    return hmac.new(key.encode('utf-8'), s.encode('utf-8'), 'MD5').hexdigest()   # key 和消息都编码成 bytes

class User:
    def __init__(self, username, password):
        self.username = username
        self.key = ''.join([chr(random.randint(48, 122)) for i in range(20)])   # 随机 key，相当于“升级版盐”
        self.password = hmac_md5(self.key, password)                            # 存 HMAC 值

def login(username, password):
    user = db[username]
    return hmac.compare_digest(user.password, hmac_md5(user.key, password))     # 恒定时间比较
```

注意这里**每个用户有自己的随机 `key`**，和上一页“每个用户随机盐”的思路一致：相同的密码，因为 key 不同，算出来的 HMAC 也不同，预计算表彻底失效。

---

## 自测

1. HMAC 和你手写的 `hash(message + salt)`，本质区别是什么？为什么说 HMAC “更标准化、更安全”？
2. `hmac.new(key, msg, digestmod)` 的三个参数分别是什么？输出长度由谁决定？
3. 登录验证时为什么不能用 `==` 比较 HMAC 值？`hmac.compare_digest` 解决了什么问题？
4. HMAC 的 key 和上一页的 salt，在“要不要保密”上有什么区别？
5. 如果你的系统需要“证明消息没被篡改，并且确实来自持有密钥的一方”，应该用哈希、HMAC 还是加密？为什么？