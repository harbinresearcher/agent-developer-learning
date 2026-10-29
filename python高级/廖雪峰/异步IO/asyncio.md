> **重要程度：高**｜**手练：需要**
> 重点：事件循环、并发任务、超时、异常和取消。

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

**第一，这个例子的协程都在主线程运行。** 一个事件循环通常在一个线程中调度任务；应用也可以使用多个线程或进程。

**第二，两个 `hello` 是并发执行的。** 两个协程都跑到 `await asyncio.sleep(1)` 时，都让出了控制权，事件循环**同时等这两个 1 秒的“IO”**，总共只花约 1 秒，不是 2 秒。

**第三，`asyncio.gather()` 会按传入顺序返回所有协程的返回值。** 上面 `L` 拿到的是 `['Bob', 'Alice']`，顺序和 `gather` 的参数顺序一致。

---

## 网络 IO

实际 HTTP 请求优先使用支持异步的客户端，并设置超时、限制并发、复用连接、检查响应状态。示例见 [[aiohttp]]。

---

## 并发任务：`TaskGroup` 与 `gather`

廖雪峰原文用的是 `asyncio.gather()`，它在绝大多数场景下够用。但有一个重要的现代升级你需要知道：**Python 3.11 引入了 `asyncio.TaskGroup`，它是 `gather` 的结构化并发替代品**。

**`gather` 的默认行为**：第一个异常会传给等待方，其他已提交任务不会因此自动取消，仍可能继续运行。使用 `return_exceptions=True` 才会把异常作为结果收集。

**`TaskGroup` 的行为**：当组内任务抛出非 `CancelledError` 的未处理异常时，`TaskGroup` 会请求取消其他任务，并等待它们结束，这就是“fail-fast”设计。

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(hello("Bob"))
        tg.create_task(hello("Alice"))
    # 出了 with 块，所有任务要么全部成功，要么有一个失败导致整组取消
```

**怎么选**：需要任务组内失败后取消其余任务并等待清理时用 `TaskGroup`；需要保留其他任务运行或收集部分失败结果时，可考虑 `gather(..., return_exceptions=True)`。取消不能撤销已经发生的外部副作用；`TaskGroup` 可能抛出 `ExceptionGroup`。

---

## ⚠️ FastAPI 里必须盯死的规矩

这页讲的是 asyncio 本身，但落到 FastAPI，有一条铁律：**避免在事件循环线程中调用耗时的阻塞操作**；短暂的普通同步计算可以直接调用。

**反面教材**：

```python
@app.get("/bad")
async def bad():
    time.sleep(3)      # 阻塞！事件循环被卡死 3 秒，所有请求排队
    return {"msg": "done"}
```

`time.sleep` 会阻塞当前事件循环线程，期间该循环无法推进其他任务；其他进程或线程不一定停摆。

**正确做法**有三种：

1. **用异步库**：`await asyncio.sleep(3)`，把控制权让出去；
2. **写普通 `def`**：FastAPI 会把普通 `def` 路由丢进线程池，不卡事件循环；
3. **如果必须在一个 `async def` 里调同步阻塞函数**，用 `await asyncio.to_thread(blocking_func, ...)` 把它扔到线程池里跑，**这是 Python 3.9+ 的推荐方式**。

异步路由可包含普通同步代码；重点是避免耗时的阻塞调用。线程池适合阻塞 IO，纯 Python CPU 密集任务通常考虑进程池。

---

## 📝 自测

1. `asyncio.run()` 做了什么？为什么说它是启动异步程序的现代标准方式？
2. `asyncio.gather()` 和 `asyncio.TaskGroup` 在错误处理上的根本区别是什么？什么场景下用哪个？
3. 在 `async def` 里写 `time.sleep(3)` 会发生什么？有哪三种修正方式？
4. `asyncio.gather()` 返回的结果顺序和什么有关？
5. 一个事件循环如何在同一线程中重叠多个 IO 等待？