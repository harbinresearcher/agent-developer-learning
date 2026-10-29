> **重要程度：中**｜**手练：建议**
> 重点：连接、字节流分帧、超时和连接关闭。

这一页是网络编程的入门基础，核心就是：用 Python 的 `socket` 库，实现“客户端主动连接、服务器被动监听并处理”的通信流程。我们按“客户端怎么连 → 服务器怎么接 → 怎么处理多个连接”来走。

---

# TCP编程 · 笔记

## 先搞清一个概念：Socket是什么

你可以把 Socket 理解成“打开了一个网络链接”。想连上另一台电脑，你需要知道它的 **IP 地址**和**端口号**，然后指定用 **TCP 协议**。

创建 Socket 就一句：

```python
import socket                                 # 导入 socket 库

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # AF_INET 表示 IPv4，SOCK_STREAM 表示 TCP 协议
```

**关键点**：`AF_INET` 是 IPv4，`SOCK_STREAM` 是面向流的 TCP。创建出来还没连上，它只是一个“准备连接”的对象。

---

## 客户端：主动连接 → 发送请求 → 接收数据 → 关闭

客户端是最简单的，就四步。

### 第一步：建立连接

```python
s.connect(('www.sina.com.cn', 80))            # 传入 (域名或IP, 端口号) 的元组，发起 TCP 连接
```

**为什么是 80 端口**：HTTP 的默认端口是 80，HTTPS 的默认端口是 443；服务也可使用其他端口。端口是否可绑定取决于系统权限、占用情况和配置。

### 第二步：发送数据

```python
s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')  # 发送 HTTP 请求，b'' 表示 bytes，\r\n 是 HTTP 规定的换行
```

**注意**：`send()` 收的是 **`bytes`**，不是字符串。你发的内容必须符合对方协议的标准，发 HTTP 就得拼 HTTP 请求头，发自定义协议就按自定义格式来。

### 第三步：接收数据

```python
buffer = []                                   # 用列表收集所有收到的数据块
while True:                                   # 因为数据可能分多次到达，所以循环收
    d = s.recv(1024)                          # 每次最多收 1024 字节
    if d:                                     # 收到了数据
        buffer.append(d)                      # 存进列表
    else:                                     # recv 返回空 bytes，表示对方关闭了连接
        break                                 # 退出循环，接收完毕
data = b''.join(buffer)                       # 把所有数据块拼成一个完整的 bytes 对象
```

**核心坑**：`recv(1024)` 不是“收满 1024 字节才返回”，而是“**最多收 1024 字节**”，可能只收到几百字节就返回了。所以**必须循环收，直到 `recv()` 返回空 `b''`**，才代表对方把数据发完了。

### 第四步：关闭连接

```python
s.close()                                     # 关闭 Socket，释放资源
```

到这一步，一次完整的客户端通信就结束了。

### 完整客户端示例（访问新浪首页）

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)   # 创建 TCP Socket
s.connect(('www.sina.com.cn', 80))                       # 连接新浪的 80 端口
s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')  # 发送 HTTP 请求

buffer = []
while True:
    d = s.recv(1024)                                     # 每次最多收 1KB
    if d:
        buffer.append(d)                                 # 攒起来
    else:
        break                                            # 收完了
data = b''.join(buffer)                                  # 拼成完整响应
header, html = data.split(b'\r\n\r\n', 1)                # 按空行拆分 HTTP 头和网页内容
print(header.decode('utf-8'))                            # 打印 HTTP 头

with open('sina.html', 'wb') as f:                       # 把网页内容写入文件
    f.write(html)
```

---

## 服务器：绑定端口 → 监听 → 接受连接 → 多线程处理

服务器比客户端复杂，因为它要**一直运行**，等着别人来连。

### 第一步：创建 Socket 并绑定端口

```python
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 创建 TCP Socket
s.bind(('127.0.0.1', 9999))                            # 绑定本机地址的 9999 端口
```

`bind()` 的参数是 `(IP, 端口)`。`127.0.0.1` 表示**只接受本机连接**，外部机器连不进来。如果想让所有机器都能连，用 `'0.0.0.0'`。

**端口选择**：低端口的权限要求取决于操作系统和配置；本地练习可选未被占用的高端口。

### 第二步：开始监听

```python
s.listen(5)                                            # 开始监听，参数 5 是等待连接队列的最大长度
print('Waiting for connection...')
```

`listen()` 之后，Socket 就开始等别人来连了。**`listen` 本身不会阻塞**，它只是告诉操作系统“我准备好了”。

### 第三步：循环接受连接，每个连接开一个新线程

```python
while True:                                            # 服务器要永远运行下去
    sock, addr = s.accept()                            # accept() 会阻塞，直到有客户端连上来，返回 (新Socket, 客户端地址)
    t = threading.Thread(target=tcplink, args=(sock, addr))  # 每来一个连接，开一个新线程去处理
    t.start()                                          # 启动线程
```

**如何并发处理连接**：`accept()` 返回的 `sock` 是**专门用来和这个客户端通信的 Socket**，和监听用的 `s` 不是同一个。如果你不开线程，直接在当前循环里处理这个客户端，**服务器就被卡住了，没法 accept 下一个连接**。线程是一种做法，也可使用线程池、异步 IO 或其他并发模型；注意连接数限制。

### 第四步：线程里处理具体通信

```python
def tcplink(sock, addr):                               # 每个连接的处理函数
    print('Accept new connection from %s:%s...' % addr)
    sock.send(b'Welcome!')                             # 先发一条欢迎消息
    while True:
        data = sock.recv(1024)                         # 收客户端发来的数据
        time.sleep(1)                                  # 模拟处理耗时
        if not data or data.decode('utf-8') == 'exit':  # 收到空数据或 exit 就退出
            break
        sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))  # 加个 Hello 再发回去
    sock.close()                                       # 关闭这个连接的 Socket
    print('Connection from %s:%s closed.' % addr)
```

**核心逻辑**：服务器收到数据 → 加工 → 发回去。`sock` 是**这个连接专属的**，不同客户端的 `sock` 互不干扰。

---

## 一个必须知道的坑：Socket 由 4 项唯一确定

原文有一句很关键的话：**一个 Socket 依赖 4 项来唯一确定——服务器地址、服务器端口、客户端地址、客户端端口**。

这意味着：**同一个服务器端口（比如 80）可以同时被无数个客户端连接**，因为每个连接的四元组都不同（客户端的 IP 和端口不一样）。绑定冲突还受 IP 地址、协议、系统及套接字选项影响，不能只看端口号。

---

## 总结：客户端和服务器的核心差异

| | **客户端** | **服务器** |
|---|---|---|
| **动作** | 主动 `connect()` | 先 `bind()` + `listen()`，再循环 `accept()` |
| **连接管理** | 只连一个服务器 | 每个客户端连接开一个新线程/进程 |
| **运行方式** | 通信完就退出 | 永远循环，直到 Ctrl+C |
| **Socket 数量** | 1 个 | 1 个监听 + N 个通信 |

---

## 自测

1. `s.recv(1024)` 是“收满 1024 字节才返回”还是“最多收 1024 字节”？为什么必须循环收？
2. 服务器为什么每来一个连接就要开一个新线程？不开会怎样？
3. `accept()` 返回的 `sock` 和监听用的 `s` 有什么区别？
4. 为什么 `127.0.0.1` 只能本机连接，而 `0.0.0.0` 可以让外部连进来？
5. 客户端 `send()` 为什么可能只发送部分字节，什么时候用 `sendall()`？参数是什么类型？服务器 `recv()` 返回的是什么类型？中间需要做什么转换？
## 代码示例的边界

上面的示例只帮助理解 API，不能当生产 HTTP 客户端：`send()` 可能只发送一部分，应检查返回值或使用 `sendall()`；TCP 需要分帧，连接和读取需设超时。例子里的单次 `recv()` 不保证恰好收到一个完整文本消息，也可能拆开 UTF-8 字节。实际 HTTP 请求优先使用成熟客户端，正确处理 HTTPS 和响应格式。

参考：[Python socket](https://docs.python.org/3/library/socket.html)。