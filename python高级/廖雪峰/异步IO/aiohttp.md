> **重要程度：中**｜**手练：建议**
> 重点：复用会话、超时、并发限制、响应关闭。

这一页是将前面学到的`asyncio`和`await`真正应用到网络请求上的实战篇。它的核心是教你如何使用**异步HTTP客户端`aiohttp`**，来实现高并发的网络请求，这正是FastAPI作为高性能后端框架所依赖的能力。

### 🎯 为什么不能用`requests`？
你在同步编程中常用的`requests`库是**同步阻塞**的。如果在FastAPI的`async def`路由中调用`requests.get()`，它会卡住整个事件循环，导致所有并发请求都排队等待，异步的优势将荡然无存。

`aiohttp`则是**原生为异步设计**的，它基于`asyncio`构建，能在等待网络响应时自动让出控制权，从而实现单线程下的高并发。

### 🧱 核心用法：`ClientSession`与`async with`
使用`aiohttp`的标准范式是**使用`ClientSession`并配合`async with`语句**。

```python
import asyncio
import aiohttp

async def fetch(url):
    # 创建一个异步会话，它内部会管理连接池
    async with aiohttp.ClientSession() as session:
        # 发起异步GET请求，并等待响应
        async with session.get(url) as response:
            # 异步读取响应文本
            return await response.text()

async def main():
    html = await fetch('https://example.com')
    print(html)

asyncio.run(main())
```
这里的关键是，`session.get()` 返回支持 `async with` 的请求上下文管理器，进入时等待响应；`response.text()` 是协程，需要 `await` 获取结果。

### 🚀 实现并发：`asyncio.gather`的用武之地
要并发请求多个URL，只需创建任务列表，交给`asyncio.gather`统一调度：

```python
async def main():
    urls = ['https://example.com', 'https://example.org', 'https://example.net']
    # 为每个URL创建一个协程任务
    tasks = [fetch(url) for url in urls]
    # 并发执行所有任务，并收集结果
    results = await asyncio.gather(*tasks)
    for i, html in enumerate(results):
        print(f'第 {i+1} 个页面长度: {len(html)}')

asyncio.run(main())
```
`asyncio.gather`会并发运行所有`fetch`协程，并发能重叠等待；总耗时受最慢请求、连接池、限流和网络等因素影响。

### ⚠️ 关键注意事项
1.  **会话（Session）要复用**：`ClientSession`内部维护了连接池，**应该在整个应用生命周期内复用一个实例**，而不是在每个请求函数里都新建。在FastAPI中，通常用`lifespan`事件在应用启动时创建`ClientSession`，并存放在`app.state`中，供所有路由共享。
2.  **必须设置超时**：aiohttp 默认存在超时；当前稳定文档列出总超时 300 秒、套接字连接超时 30 秒，具体取决于版本。实际项目应根据服务需求在会话或请求上明确设置超时。参考：[aiohttp 超时](https://docs.aiohttp.org/en/stable/client_quickstart.html#timeouts)。
    ```python
    # 全局超时：总时长30秒
    timeout = aiohttp.ClientTimeout(total=30)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        # 也可以在单次请求中覆盖
        async with session.get(url, timeout=10) as response:
            ...
    ```
    超时设置需要谨慎，过短会误杀正常但稍慢的请求，过长则失去保护意义。
3.  **异常处理要分层**：网络请求可能因多种原因失败，需要分别捕获。`aiohttp.ClientError`是大多数客户端错误的基类，`asyncio.TimeoutError`则专门表示超时。
    ```python
    try:
        async with session.get(url) as response:
            response.raise_for_status() # 对4xx/5xx状态码抛出异常
            return await response.json()
    except asyncio.TimeoutError:
        # 处理超时
    except aiohttp.ClientResponseError as e:
        # 处理HTTP状态码错误 (4xx, 5xx)
    except aiohttp.ClientError:
        # 处理其他网络层错误（连接失败、DNS错误等）
    ```

### 🔄 与FastAPI生态：`aiohttp` vs `httpx`
`aiohttp`是纯粹的异步库，性能强劲。但在FastAPI生态中，你可能会更常遇到**`httpx`**。

- **`aiohttp`**：**仅支持异步**，专为高并发设计，性能是三者中的顶尖水平，适合对吞吐量要求极高的场景。
- **`httpx`**：**同时支持同步和异步**，API设计上兼容`requests`，迁移成本低。更重要的是，**FastAPI官方推荐使用`httpx`**（它的`TestClient`就基于`httpx`构建），并且OpenAI等主流SDK也选择了`httpx`。

**选择建议**：
- 如果你的应用**完全基于异步**，且追求极致性能，`aiohttp`是可靠的选择。
- 如果你希望**统一同步和异步的代码风格**，或者需要与FastAPI的测试生态无缝集成，**`httpx`的`AsyncClient`是更推荐的现代方案**。

### 💎 总结
`aiohttp`是你在`asyncio`之上进行高效网络请求的利器。核心是**复用`ClientSession`**、**用`async with`管理资源**、**通过`asyncio.gather`实现并发**、**务必设置超时并分层处理异常**。在FastAPI项目中，`httpx`的`AsyncClient`也是一个值得优先考虑的现代替代方案。

这一页的内容是将异步理论付诸实践的关键一步。学完它，你对FastAPI如何处理高并发I/O请求的理解就完整了。