这页是整个异步 IO 系列的核心，原文把 `asyncio` 从基本模型到实战代码都过了一遍。咱们顺着**“asyncio 是什么 → 基本写法 → 怎么并发 → 实战抓网页 → 现代 Python 的关键升级”**这条线走。

---

## 🎯 先定性：asyncio 是“消息循环”的标准实现

原文开篇定得很准：**`asyncio` 是 Python 3.4 引入的标准库，内置了对异步 IO 的支持，它的编程模型就是一个消息循环**。`asyncio` 模块内部实现了 **EventLoop（事件循环）**，你把需要执行的协程扔进去，它就替你调度，实现异步 IO。

这里有一个关键的历史信息：Python 3.4 时代用的是 `@asyncio.coroutine` 和 `yield from` 来标记协程，**Python 3.5 开始引入了 `async` 和 `await` 语法，让代码更简洁易读**。你现在学的、以及 FastAPI 里用的，全是 `async/await` 这套新语法。

---

## 🧱 最小例子：`asyncio.run()` + `await`

原文给了一个 `Hello world` 级别的例子，这是所有 asyncio 代码的骨架：

```python
import asyncio

async def hello():
    print("Hello world!")
    await asyncio.sleep(1)   # 模拟一个耗时 1 秒的 IO 操作
    print("Hello again!")

asyncio.run(hello())         # 启动事件循环，运行 hello()
```

**`asyncio.run()` 是入口**，它负责创建事件循环、运行传入的协程、结束后关闭循环。现代 Python 里，**这是启动异步程序的标准方式**，不需要手动去碰事件循环。

**`await asyncio.sleep(1)` 发生了什么**：`asyncio.sleep()` 本身是一个协程，遇到它时，当前协程会**暂停**，把控制权交还给事件循环。事件循环发现主线程没有别的事可做（因为只有一个 `hello`），就静静等 1 秒，然后恢复执行。原文对此的描述是：**`await` 让线程不等待 `asyncio.sleep()`，而是直接中断并执行下一个消息循环**。

**但现在只有一个协程，看不出并发的威力。**

---

## 🚀 怎么并发：`asyncio.gather()` 同时跑多个

原文把 `hello()` 改成接收 `name` 参数，然后用 `asyncio.gather()` 同时调度多个：

```python
import asyncio, threading

async def hello(name):
    print("Hello %s! (%s)" % (name, threading.current_thread()))
    await asyncio.sleep(1)
    print("Hello %s again! (%s)" % (name, threading.current_thread()))
    return name

async def main():
    L = await asyncio.gather(
        hello("Bob"),
        hello("Alice")
    )
    print(L)

asyncio.run(main())
```

**输出会是**：

```
Hello Bob! (<_MainThread>)
Hello Alice! (<_MainThread>)
(等待约1秒)
Hello Bob again! (<_MainThread>)
Hello Alice again! (<_MainThread>)
['Bob', 'Alice']
```

这里有几个必须钉死的结论：

**第一，所有协程都在同一个线程里跑。** `threading.current_thread()` 打印出来全是 `MainThread`，**asyncio 是单线程的并发**，不是多线程。

**第二，两个 `hello` 是并发执行的。** 两个协程都跑到 `await asyncio.sleep(1)` 时，都让出了控制权，事件循环**同时等这两个 1 秒的“IO”**，总共只花约 1 秒，不是 2 秒。

**第三，`asyncio.gather()` 会按传入顺序返回所有协程的返回值。** 上面 `L` 拿到的是 `['Bob', 'Alice']`，顺序和 `gather` 的参数顺序一致。

---

## 🌐 实战：并发抓取三个网站的首页

原文用 `asyncio.open_connection()` 演示了真正的网络 IO 并发。核心是**不用任何第三方库，直接用 asyncio 内置的 TCP 连接**：

```python
import asyncio

async def wget(host):
    print(f"wget {host}...")
    reader, writer = await asyncio.open_connection(host, 80)   # 异步建立 TCP 连接
    header = f"GET / HTTP/1.0\r\nHost: {host}\r\n\r\n"
    writer.write(header.encode("utf-8"))
    await writer.drain()                                       # 异步刷新写缓冲区

    while True:
        line = await reader.readline()                        # 异步读一行
        if line == b"\r\n":
            break
        print("%s header > %s" % (host, line.decode("utf-8").rstrip()))

    writer.close()
    await writer.wait_closed()                                # 异步等待关闭
    print(f"Done {host}.")

async def main():
    await asyncio.gather(
        wget("www.sina.com.cn"),
        wget("www.sohu.com"),
        wget("www.163.com")
    )

asyncio.run(main())
```

**执行效果**：三个 `wget` 几乎同时开始，**打印顺序是交错的**，因为每个 `await` 在等待网络返回时，都把控制权让了出来，事件循环立刻切去处理另一个 `wget`。**三个网站的首页是真正同时抓取的，总共花的时间约等于最慢的那个网站**，而不是三个时间相加。

**这段代码的价值在于**：它让你看到，**`asyncio` 不依赖多线程，就能把多个网络 IO 的等待时间重叠起来**。你把 `asyncio.open_connection` 换成 `aiohttp` 或 `httpx` 的异步客户端，逻辑完全一样。

---

## 🔄 现代 Python 的关键升级：`TaskGroup` 取代 `gather`

廖雪峰原文用的是 `asyncio.gather()`，它在绝大多数场景下够用。但有一个重要的现代升级你需要知道：**Python 3.11 引入了 `asyncio.TaskGroup`，它是 `gather` 的结构化并发替代品**。

**`gather` 的隐患**：如果 `gather` 里的某个协程抛了异常，**其他协程的错误可能会被静默忽略**，调试时很难发现。

**`TaskGroup` 的行为**：**只要组内任何一个任务抛出未处理的异常，`TaskGroup` 会立刻取消组内其他所有任务**，这就是“fail-fast”设计。

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(hello("Bob"))
        tg.create_task(hello("Alice"))
    # 出了 with 块，所有任务要么全部成功，要么有一个失败导致整组取消
```

**怎么选**：如果你的并发任务是“要么全部成功，要么一个失败就整体放弃”，用 `TaskGroup`；如果你需要收集所有结果、即使部分失败也想要其他成功的结果，用 `gather`。**FastAPI 的生态里，`gather` 依然非常常见**，但新代码可以逐步向 `TaskGroup` 迁移。

---

## ⚠️ FastAPI 里必须盯死的规矩

这页讲的是 asyncio 本身，但落到 FastAPI，有一条铁律：**在 `async def` 路由里，绝对不能调用阻塞的同步代码**。

**反面教材**：

```python
@app.get("/bad")
async def bad():
    time.sleep(3)      # 阻塞！事件循环被卡死 3 秒，所有请求排队
    return {"msg": "done"}
```

`time.sleep` 不是 `await` 的，事件循环**没法在等待它的时候切去处理其他请求**，整个应用停摆。

**正确做法**有三种：

1. **用异步库**：`await asyncio.sleep(3)`，把控制权让出去；
2. **写普通 `def`**：FastAPI 会把普通 `def` 路由丢进线程池，不卡事件循环；
3. **如果必须在一个 `async def` 里调同步阻塞函数**，用 `await asyncio.to_thread(blocking_func, ...)` 把它扔到线程池里跑，**这是 Python 3.9+ 的推荐方式**。

**一句话**：**`async def` 里只放支持 `await` 的东西，拿不准就写 `def`。**

---

## 📝 自测

1. `asyncio.run()` 做了什么？为什么说它是启动异步程序的现代标准方式？
2. `asyncio.gather()` 和 `asyncio.TaskGroup` 在错误处理上的根本区别是什么？什么场景下用哪个？
3. 在 `async def` 里写 `time.sleep(3)` 会发生什么？有哪三种修正方式？
4. `asyncio.gather()` 返回的结果顺序和什么有关？
5. 为什么说 asyncio 的并发是“单线程的并发”？它靠什么机制把多个 IO 的等待时间重叠起来？