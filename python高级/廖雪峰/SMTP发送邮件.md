
---

## 先聊痛点：你平时是怎么发邮件的

你平时发邮件，是打开网页或客户端，填好收件人、主题、正文，点发送，剩下的交给邮箱服务商。

`SMTP` 就是这套动作的底层协议。Python 的 `smtplib` 让你**用代码把“填内容、点发送”这件事自动化**，比如定时把报表发到邮箱、系统出错时自动发警报。

核心工具就两个模块：`email` 负责构造邮件，`smtplib` 负责把邮件发出去。

---

## 最小例子：先发一封“裸”的纯文本

先不求完整，把一封最简陋的邮件发出去。

**构造邮件正文**用 `MIMEText`：

```python
from email.mime.text import MIMEText

msg = MIMEText('hello, send by Python...', 'plain', 'utf-8')  # 正文、子类型、编码
```

第一个参数是正文；第二个 `'plain'` 表示纯文本；第三个 `'utf-8'` 保证中文兼容。

**发送**用 `smtplib`：

```python
import smtplib

server = smtplib.SMTP(smtp_server, 25)  # 连接 SMTP 服务器，默认端口 25
server.set_debuglevel(1)                # 打印和服务器交互的所有信息，调试用
server.login(from_addr, password)       # 登录
server.sendmail(from_addr, [to_addr], msg.as_string())  # 发送，收件人必须是列表
server.quit()                           # 断开连接
```

`sendmail` 的第二个参数是一个**列表**，因为可以一次发给多人；`msg.as_string()` 把 `MIMEText` 对象转成字符串。

**注意**：这样发出去的邮件，收件人看到的是**没有主题、没有友好显示名、甚至提示“不在收件人中”**，因为 `From`、`To`、`Subject` 这些信息没有加进去。

---

## 核心干货：一封“完整”的邮件必须有三样东西

邮件头里的 `From`、`To`、`Subject` 不是靠 SMTP 协议传的，而是**写在邮件正文里的**。所以必须把它们加到 `MIMEText` 对象上，才是一封完整的邮件。

### 难点：中文名字不能直接塞进邮件头

如果你写 `msg['From'] = 'Python爱好者 <xxx@xx.com>'`，中文部分在传输中会乱码，因为邮件头默认只认 ASCII。

**标准解法**：用 `Header` 对象对中文名字编码，再用 `formataddr` 拼成合法格式。原文写了一个辅助函数：

```python
from email.header import Header
from email.utils import parseaddr, formataddr

def _format_addr(s):
    name, addr = parseaddr(s)                        # 拆出名字和地址
    return formataddr((Header(name, 'utf-8').encode(), addr))  # 名字用 utf-8 编码，再拼回去
```

然后这样设置三个头：

```python
msg['From'] = _format_addr('Python爱好者 <%s>' % from_addr)
msg['To'] = _format_addr('管理员 <%s>' % to_addr)      # 注意：To 是字符串，多人用逗号分隔
msg['Subject'] = Header('来自SMTP的问候……', 'utf-8').encode()
```

`Header(...).encode()` 会把中文名字编码成 `=?utf-8?b?...?=` 的格式，邮件客户端看到这个格式就会正确解码显示。

---

## 现代环境下的几个关键补充

原文用的 `SMTP(smtp_server, 25)` 在今天的公网邮箱服务上已经**基本行不通**，因为主流服务商都要求加密连接，并且登录密码不是你的邮箱密码。

### 端口和加密方式：465 / 587 二选一

- **465 端口 + `SMTP_SSL`**：连接建立时就加密（隐式 TLS），代码是 `smtplib.SMTP_SSL(smtp_server, 465)`。
- **587 端口 + `SMTP` + `starttls()`**：先明文连接，再升级为加密（显式 TLS），代码是 `server = smtplib.SMTP(smtp_server, 587); server.starttls()`。

**推荐用 465 + `SMTP_SSL`**，代码更直接，不容易出错。25 端口在公网上大多被运营商封禁，只适合内网测试。

### 密码不是邮箱密码，是“授权码”

QQ 邮箱、163 等主流服务商都要求使用 **SMTP 授权码**登录，而不是网页登录密码。授权码需要在邮箱设置里单独生成。

```python
password = input('Authorization Code: ')  # 输入的是授权码，不是邮箱密码
```

---

## 易踩坑

**坑一：`sendmail` 的收件人必须是列表。** `[to_addr]` 不能写成 `to_addr`，否则会被当成三个独立的字符去投递。

**坑二：`msg['To']` 是字符串，不是列表。** 多个收件人用逗号分隔，比如 `'a@x.com, b@y.com'`，不要传列表。

**坑三：忘记 `Header` 编码，中文主题和名字全变乱码。** 只要是中文，`From`、`To` 的名字部分和 `Subject` 都必须过一遍 `Header(...).encode()`。

**坑四：用 25 端口连公网邮箱，直接超时或被拒。** 换成 465 + `SMTP_SSL`，或者 587 + `starttls()`。

**坑五：把邮箱登录密码当 `password` 传。** 绝大多数服务商会直接报认证失败，必须用授权码。

---

## 一句话收束

**用 `email.mime` 构造邮件，用 `Header` 处理中文头，用 `smtplib` 登录并发送；公网发信优先用 465 端口 + `SMTP_SSL`，密码用授权码。** 这封邮件就能完整地送到收件箱里。