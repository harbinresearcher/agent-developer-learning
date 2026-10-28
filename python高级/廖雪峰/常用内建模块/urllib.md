这页的核心其实就一句话：**`urllib` 让你用 Python 代码扮演浏览器，向服务器发送 HTTP 请求并拿回响应**。原文只讲了 GET 和 POST 两种最基础的用法，但其中包含了几个必须掌握的细节和坑。

---

# urllib · 笔记

## 先聊痛点：为什么需要它

你平时在浏览器地址栏输入网址、按回车，浏览器帮你做了“发送 HTTP 请求、接收响应、渲染页面”这一整套动作。**`urllib` 就是把“发送请求、接收响应”这件事，从浏览器手里拿出来，交给你用代码控制**。这样你才能做爬虫、调 API、模拟登录这些浏览器做不到或做起来很麻烦的事。

但原文演示的只是最基础的 `urllib.request`，实际上 `urllib` 是一个**包**，里面装着好几个模块，分工不同：

- **`urllib.request`**：发送请求、获取响应，原文的主角；
- **`urllib.parse`**：处理 URL，比如编码查询参数、拼接 URL；
- **`urllib.error`**：定义 `urllib.request` 会抛出的异常类，比如 `URLError`、`HTTPError`；
- **`urllib.robotparser`**：解析网站的 `robots.txt`，判断你能不能爬某个页面。

日常用得最多的是前两个，后两个在你需要处理异常或做合规爬虫时才会碰。

---

## GET 请求：最直接的抓取方式

`urlopen()` 是发送请求最直接的工具。最简单的用法就是传入一个 URL 字符串，它会返回一个**响应对象**，这个对象**像文件对象一样**，你可以对它调用 `.read()`、`.status`、`.getheaders()` 等。

```python
from urllib import request

with request.urlopen('https://api.douban.com/v2/book/2129650') as f:
    data = f.read()                          # 读出响应体，是 bytes
    print('Status:', f.status, f.reason)     # 状态码和原因短语，比如 200 OK
    for k, v in f.getheaders():              # 遍历响应头
        print('%s: %s' % (k, v))
    print('Data:', data.decode('utf-8'))     # 把 bytes 解码成字符串才能读
```

**三个关键点要钉死**：

第一，**`.read()` 返回的是 `bytes`**，不是 `str`。要拿到能读的文本，必须 `.decode('utf-8')`。服务器返回的编码可能不是 UTF-8，但绝大多数现代 API 都是。

第二，**`f.status` 和 `f.reason` 给你 HTTP 状态码**，`200 OK` 表示成功。**不是所有请求失败都会抛异常**，有些 4xx 错误需要你自己检查状态码，或者捕获 `HTTPError`（后面讲）。

第三，**`with` 语句在这里同样适用**，它会自动帮你关闭连接，和操作文件一个道理。

---

## 伪装浏览器：`Request` 对象 + `add_header`

直接用 `urlopen(URL)` 发出去的请求，`User-Agent` 是 `Python-urllib/x.y`，**服务器一眼就能看出来“这不是真人浏览器”**，很多网站会直接拒绝或返回不同的内容。要模拟浏览器，就必须用 `Request` 对象，**往里面加 HTTP 头**。

```python
from urllib import request

req = request.Request('http://www.douban.com/')          # 创建 Request 对象
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; ...)')   # 伪装成 iPhone 浏览器
with request.urlopen(req) as f:                          # 把 Request 对象交给 urlopen
    print(f.read().decode('utf-8'))
```

**关键区别**：`urlopen()` 既可以接受**字符串 URL**，也可以接受**`Request` 对象**。只有用 `Request` 对象，你才能**在发送前修改请求头**。`add_header()` 是往这个 Request 对象上“贴标签”的动作，可以多次调用加多个头。

---

## POST 请求：把数据放进 `data` 参数

GET 是把参数拼在 URL 后面，POST 是把参数**放在请求体里**。用 `urllib` 发 POST 的关键就一条：**把编码后的 `data` 以 `bytes` 形式传给 `urlopen()`**，它就会自动用 POST 方法。

```python
from urllib import request, parse

login_data = parse.urlencode([                 # 把键值对编码成查询字符串
    ('username', email),
    ('password', passwd),
    ('entry', 'mweibo'),
])
req = request.Request('https://passport.weibo.cn/sso/login')
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; ...)')   # POST 通常也要伪装
with request.urlopen(req, data=login_data.encode('utf-8')) as f:   # data 必须是 bytes
    print(f.read().decode('utf-8'))
```

**关键点**：`parse.urlencode()` 负责把键值对列表编码成 `username=xxx&password=xxx` 这种形式。**`urlencode()` 返回的是 `str`，但 `urlopen` 的 `data` 参数要求 `bytes`**，所以必须再 `.encode('utf-8')`。

---

## 坑一：`urlopen` 失败时抛什么异常

原文没有展开，但这是实际写代码时绕不开的。`urlopen` 失败时抛出的异常主要有两个，**必须分清**：

**`urllib.error.URLError`**：这是**所有网络层错误的基类**，比如 DNS 解析失败、连接被拒绝、超时、URL 格式错误。

**`urllib.error.HTTPError`**：这是 `URLError` 的**子类**，专门表示**服务器返回了 HTTP 错误状态码**（4xx、5xx），比如 `403 Forbidden`、`404 Not Found`。

**关键规则**：**如果要同时捕获两者，`HTTPError` 必须写在 `URLError` 前面**，因为子类在前才能被优先匹配。

```python
from urllib import request, error

try:
    with request.urlopen('https://example.com/not-exist') as f:
        pass
except error.HTTPError as e:          # 先捕子类：HTTP 错误，有状态码
    print('HTTP Error:', e.code, e.reason)
except error.URLError as e:           # 后捕父类：网络层错误
    print('URL Error:', e.reason)
```

---

## 坑二：`data` 参数决定 GET 还是 POST

这是一个很隐蔽的行为：**`urlopen()` 的 `data` 参数，如果传了 `bytes`，请求方法就是 POST；如果不传（默认 `None`），就是 GET**。你不需要显式指定方法，**有没有 `data` 就是开关**。

```python
# GET：不传 data
request.urlopen('https://example.com/search?q=python')

# POST：传 data，必须是 bytes
request.urlopen('https://example.com/login', data=b'username=admin&password=123')
```

---

## 坑三：响应体编码不一定是你以为的那个

`.read()` 返回 `bytes`，你需要 `.decode()` 才能得到字符串。但**解码时用的编码，必须是服务器实际返回的编码**，不一定是 UTF-8。你可以从响应头 `Content-Type` 里看 `charset` 字段：

```python
content_type = f.getheader('Content-Type')   # 比如 'application/json; charset=utf-8'
# 从里面提取 charset，用它来 decode，而不是无脑 utf-8
```

大多数现代 API 都是 UTF-8，但**遇到乱码时，第一反应应该是去检查响应头里的 `charset`**，而不是怀疑数据本身。

---

## 一句话收束

**`urllib.request` 用 `urlopen()` 发请求，返回一个像文件一样的响应对象；用 `Request` 对象加 `add_header` 伪装浏览器；`data` 参数传 `bytes` 就是 POST，不传就是 GET；失败时先捕 `HTTPError` 再捕 `URLError`；读响应记得 `.decode()`。**

---

## 自测

1. `urlopen()` 返回的响应对象，`.read()` 返回的是什么类型？要拿到可读文本需要做什么？
2. 为什么直接用 `urlopen(URL)` 容易被网站识别为爬虫？`Request` 对象在这里起什么作用？
3. `urlopen()` 的 `data` 参数和 GET/POST 方法之间是什么关系？
4. `URLError` 和 `HTTPError` 是什么关系？同时捕获时为什么 `HTTPError` 要写在前面？
5. `urlencode()` 返回 `str` 还是 `bytes`？`urlopen` 的 `data` 参数需要什么类型？中间差了什么操作？