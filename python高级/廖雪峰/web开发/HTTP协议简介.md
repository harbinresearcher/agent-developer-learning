> **重要程度：高**｜**手练：需要**
> 重点：请求响应、方法、状态码、Header 与超时。


## 先定性：HTTP 是“请求-响应”的通信规则

HTTP 是浏览器和服务器之间传输数据的协议，核心就是**客户端先发请求，服务器再回响应**。HTML 定义了网页长什么样，而 HTTP 负责把 HTML 在网络上传输过去。

## 报文长什么样：请求与响应的解剖

原文教我们在 Chrome 的 Network 面板里看报文。一条 HTTP 请求通常包含四部分：**请求行、请求头、空行、请求体**；响应也类似：**状态行、响应头、空行、响应体**。

### 请求：客户端对服务器说的话

请求的第一行是**请求行**，例如 `GET / HTTP/1.1`。`GET` 表示读取请求，`/` 是路径，`HTTP/1.1` 是协议版本。后续每一行都是**请求头**，例如 `Host: www.sina.com.cn`，告诉服务器你想访问哪个网站。

### 响应：服务器对客户端的答复

响应的第一行是**状态行**，例如 `200 OK`，200 表示成功；`404 Not Found` 表示资源不存在；`500 Internal Server Error` 表示服务器内部出错。响应头里最重要的是 `Content-Type`，例如 `text/html` 表示返回的是网页。浏览器依赖这个字段判断内容类型，而不是靠 URL 后缀。

---

## FastAPI 帮你做了什么

FastAPI 扮演的是“翻译官”的角色：它把 HTTP 请求翻译成 Python 能识别的参数，把你返回的 Python 对象再翻译成 HTTP 响应。

### 请求方法：用装饰器声明

在 FastAPI 中，你不需要手动解析请求行，**用装饰器声明**即可：

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/items")    # 声明处理 GET 请求
def read_items():
    return {"msg": "这是GET"}

@app.post("/items")   # 声明处理 POST 请求
def create_item():
    return {"msg": "这是POST"}
```

常见的请求方法包括：**GET**（查询资源）、**POST**（创建资源）、**PUT**（更新资源）、**DELETE**（删除资源）。

### 请求头与响应头：按需处理

FastAPI 会自动解析请求头，你也可以在函数里直接注入 `Response` 对象来设置响应头：

```python
from fastapi import Response

@app.get("/headers")
def get_headers(response: Response):
    response.headers["X-Custom-Header"] = "Hello"   # 设置自定义响应头
    return {"message": "ok"}
```

`Content-Type` 默认由 FastAPI 根据返回内容自动设置（通常返回 JSON 时就是 `application/json`），不需要手动操心。

### 状态码：默认与自定义

FastAPI 默认返回 **200 OK**。如果要指定其他状态码，在装饰器里加 `status_code` 参数：

```python
@app.post("/items", status_code=201)   # 创建成功返回 201
def create_item():
    return {"msg": "created"}
```

如果请求的资源不存在，可以主动抛出 `HTTPException` 来返回 404：

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id not in db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item_id": item_id}
```

## 和 FastAPI 的对应关系（速查表）

| HTTP 概念 | 浏览器里看到的 | FastAPI 代码中的体现 |
|---|---|---|
| 请求方法 | `GET /path` | `@app.get("/path")` |
| 请求头 | `Header: value` | 函数参数 `user_agent: str = Header(None)` |
| 请求体 | POST 的正文（JSON） | Pydantic 模型作为参数 `item: Item` |
| 状态码 | `200 OK` | 装饰器 `status_code=200` 或 `HTTPException` |
| 响应头 | `Content-Type: ...` | 注入 `Response` 对象后 `response.headers[...]` |
| 响应体 | HTML / JSON | 函数 `return` 的字典 / Pydantic 对象 |

## 几个必须盯死的点

**① `Content-Type` 决定浏览器怎么处理响应。** 不要依赖 URL 后缀。FastAPI 返回字典时自动设为 `application/json`，这就是前后端分离项目里最标准的做法。

**② `GET` 不应该带请求体。** GET 的语义是“读取”，参数应该放在 URL 的查询字符串里；`POST`、`PUT`、`PATCH` 等方法都可以携带请求体。

**③ 状态码是给机器看的，`detail` 是给人看的。** 前端会根据 `200`、`404`、`422` 等状态码做不同的逻辑，`HTTPException` 的 `detail` 字段是给开发者看的错误说明。

## 自测

1. 一个 HTTP 请求由哪几部分组成？`GET / HTTP/1.1` 这行分别代表什么？
2. `Content-Type` 为什么比 URL 后缀更靠谱？浏览器靠什么判断该显示网页还是图片？
3. FastAPI 中，`@app.get` 和 `@app.post` 分别对应 HTTP 的什么语义？GET 请求为什么不应该带请求体？
4. 想在 FastAPI 中返回 201 状态码，有哪几种写法？
5. `HTTPException(status_code=404, detail="...")` 和直接 `return` 一个字典有什么区别？