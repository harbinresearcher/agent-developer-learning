> **重要程度：中**｜**手练：建议**
> 重点：标签、表单、语义化与转义。

# HTML 简介 · 笔记

## 先定性：HTML 是网页的“骨架”

原文开篇讲得很清楚：网页就是 HTML，它用一套 **Tag（标签）** 来描述内容。一个最简单的 HTML 长这样：

```html
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
  </body>
</html>
```

`<head>` 里放的是给浏览器看的信息（比如标题），`<body>` 里放的才是页面上真正显示的内容。注意，**别把 HTML 的 `<head>`/`<body>` 和 HTTP 的 Header/Body 搞混了**，它们只是名字像，完全是两回事。

原文还顺带提了 **CSS** 和 **JavaScript**：CSS 控制“长什么样”（颜色、大小、阴影），JavaScript 控制“能干什么”（点击、交互）。但对现在的你来说，知道有这三层分工就够了，不用陷进去。

---

## FastAPI 默认返回的是 JSON，不是 HTML

这是从 HTTP 那页顺下来的最关键认知。你在 FastAPI 里写 `return {"msg": "hello"}`，FastAPI 会自动把它转成 JSON，并把 `Content-Type` 设成 `application/json`。**浏览器收到 JSON，只会把它当纯文本显示，不会渲染成页面。**

想让浏览器渲染 HTML，**必须显式告诉 FastAPI：“这次我要返回的是 HTML”**。有两种方式，对应两种场景。

---

## 方式一：`HTMLResponse`——适合返回“一小段固定的 HTML”

如果你的 HTML 很简单，直接写在 Python 代码里就行，用 `HTMLResponse` 包一下：

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse   # 导入 HTMLResponse

app = FastAPI()

@app.get("/items/", response_class=HTMLResponse)   # 声明这个路由返回 HTML
async def read_items():
    return """
    <html>
        <body>
            <h1>Look ma! HTML!</h1>
        </body>
    </html>
    """
```

`response_class=HTMLResponse` 的作用是**把响应的 `Content-Type` 从默认的 `application/json` 改成 `text/html`**，浏览器看到这个类型才会去渲染页面。你也可以直接在函数里 `return HTMLResponse(content=html_string)`，效果一样。

**什么时候用**：就一两个简单的页面，或者你只是想在 API 根路径放个“欢迎页”或“健康检查页”。

---

## 方式二：Jinja2 模板——适合“正经的页面”

原文里有一句话点破了痛点：**如果一个页面有几百上千行 HTML，你把它硬塞在 Python 字符串里，不仅难写，而且根本没法维护**。这时候就要用**模板引擎**。

FastAPI 默认支持 **Jinja2**，它是 Flask 也在用的同一套模板引擎。核心用法三步：

**第一步：准备模板文件**。在项目里建一个 `templates/` 目录，里面放 `item.html`：

```html
<html>
  <body>
    <h1>Item ID: {{ id }}</h1>   <!-- 双大括号是 Jinja2 的占位符 -->
  </body>
</html>
```

**第二步：在 Python 里配置模板目录**：

```python
from fastapi.templating import Jinja2Templates

templates = Jinja2Templates(directory="templates")   # 告诉 FastAPI 模板文件在哪
```

**第三步：在路由里渲染并返回**：

```python
from fastapi import Request
from fastapi.responses import HTMLResponse

@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    return templates.TemplateResponse(
        request=request,
        name="item.html",
        context={"id": id}   # 把 Python 变量塞进模板，供 {{ id }} 使用
    )
```

`TemplateResponse` 会**读取 `item.html`，把 `context` 里的变量替换掉 `{{ }}` 占位符，然后把最终生成的 HTML 字符串返回**，同时 `Content-Type` 自动设为 `text/html`。

**关键细节**：`TemplateResponse` 的第一个参数必须是 `request` 对象，**即使你在模板里根本不用它，也得传**，因为 Jinja2 的部分功能依赖它。

---

## 两种方式怎么选

| 场景 | 用什么 |
|---|---|
| 一个静态的欢迎页、健康检查页 | `HTMLResponse`，直接拼字符串 |
| 页面结构复杂、需要根据数据动态变化 | **Jinja2 模板**，把 HTML 和 Python 逻辑分离 |
| 前后端分离项目（Vue/React 做前端） | **两者都不用**，FastAPI 只返回 JSON |

**前后端分离的项目里，FastAPI 根本不需要返回 HTML**，它只负责提供 JSON API，页面渲染交给前端框架。你学这两种方式，是为了应对“小工具后台”或“服务端渲染的简单页面”这类场景。

---

## 几个必须盯死的坑

**① `return` 一个 HTML 字符串，不指定 `response_class`，浏览器会显示转义后的代码。** FastAPI 默认将返回的字符串作为 JSON 响应序列化，浏览器不会将它作为 HTML 页面渲染。**必须显式声明 `HTMLResponse`**。

**② 模板路径写错，直接报 `TemplateNotFound`。** `Jinja2Templates(directory="templates")` 里的路径是**相对于你启动 FastAPI 时的工作目录**，不是相对于 Python 文件。路径不对，模板加载不到，路由直接 500。

**③ `context` 不传 `request`，部分 Jinja2 特性会报错。** `TemplateResponse` 的 `request` 参数是必填的，漏了会直接报错。

**④ 不要把用户输入直接拼进 HTML 字符串。** 如果你用 `HTMLResponse` 手拼字符串，用户输入里带 `<script>` 标签，就会造成 **XSS 攻击**。使用 Jinja2 时要确认 HTML 模板启用了自动转义；不同输出上下文还需要相应处理，不能把自动转义当成所有 XSS 场景的完整防护。

---

## 自测

1. FastAPI 默认返回的 `Content-Type` 是什么？想让浏览器渲染 HTML，必须改成什么？
2. `response_class=HTMLResponse` 和直接 `return HTMLResponse(...)`，效果上有什么区别？
3. Jinja2 模板里 `{{ id }}` 和 `{% if ... %}` 分别是什么语法？`context` 字典里的 key 和模板里的变量名是什么关系？
4. 为什么 `TemplateResponse` 必须传 `request` 对象？
5. 用 `HTMLResponse` 手拼字符串返回页面，有什么安全隐患？Jinja2 是怎么帮你规避这个隐患的？