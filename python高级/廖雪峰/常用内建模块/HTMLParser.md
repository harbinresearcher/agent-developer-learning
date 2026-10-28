这页的原文核心极短，但你把它和上一页的XML解析（SAX）放在一起看，立马就能通。**HTMLParser本质上就是一套专门用来对付“HTML这种不严格语法”的事件驱动解析器**。咱们按“为什么不能用XML的套路 → 它怎么工作 → 关键方法怎么用 → 几个必须避开的坑”这条线走。

---

# HTMLParser · 笔记

## 先聊痛点：HTML是XML的“松散亲戚”

原文开篇点得很准：**HTML本质上是XML的子集，但语法没有XML那么严格**。XML要求标签必须闭合、属性必须带引号、大小写敏感；而HTML里 `<br>` 可以不闭合，`<img src=x>` 不写引号也能跑，标签大小写混着写浏览器照样认。

你拿上一页的SAX去解HTML，碰到这些“不严格”的写法，SAX会直接懵——**因为它默认你给的是一份格式良好的XML**。所以Python专门提供了 `html.parser` 模块里的 `HTMLParser` 类，**它的设计目标就是“能容忍HTML的各种不规矩”**。

---

## 核心机制：和SAX一模一样，事件驱动 + 重写回调

`HTMLParser` 的工作方式和SAX**完全是同一个模型**：你继承它，重写几个 `handle_xxx` 方法，然后把HTML字符串喂给它，它扫到对应位置就自动调你写的方法。

骨架长这样：

```python
from html.parser import HTMLParser     # 从标准库导入 HTMLParser 类
from html.entities import name2codepoint  # 后面处理特殊字符要用到

class MyHTMLParser(HTMLParser):        # 继承 HTMLParser
    def handle_starttag(self, tag, attrs):    # 读到开始标签时触发
        print('<%s>' % tag)                # tag 是标签名，attrs 是属性列表
    def handle_endtag(self, tag):             # 读到结束标签时触发
        print('%s>' % tag)
    def handle_data(self, data):              # 读到标签之间的文本时触发
        print(data)
    def handle_comment(self, data):           # 读到注释时触发
        print('<!--', data, '-->')
    def handle_entityref(self, name):         # 读到 &name; 这种命名实体时触发
        print('&%s;' % name)
    def handle_charref(self, name):           # 读到 &#123; 这种数字实体时触发
        print('&#%s;' % name)
```

**你只需要写一个类，继承 `HTMLParser`，然后重写你关心的那几个 `handle_` 方法**。不关心的不写也行，基类里有默认的空实现，不会报错。

---

## 启动解析：`feed()` 可以一块一块喂

解析器创建好之后，调 `feed()` 把HTML数据喂进去：

```python
parser = MyHTMLParser()                # 创建解析器实例
parser.feed('<html><body><h1>Hello</h1></body></html>')   # 喂入HTML，回调在这行里触发
```

原文专门强调了一个关键行为：**`feed()` 可以多次调用，不一定一次把整个HTML字符串都塞进去，可以一部分一部分塞进去**。这意味着你可以从网络流里边读边喂，不用等整个页面下载完再解析。和SAX的流式处理是一个道理。

---

## 方法逐个拆：每个 `handle_` 拿到什么

### `handle_starttag(tag, attrs)`：开始标签

`tag` 是标签名，**字符串**，比如 `'div'`、`'a'`。`attrs` 是一个 **`(属性名, 属性值)` 元组组成的列表**，不是字典。比如 `<a href="/python" class="link">` 会给你：

```python
attrs == [('href', '/python'), ('class', 'link')]
```

**为什么是列表不是字典**：因为HTML允许同一个属性出现多次（虽然不规范但现实中存在），列表能保留重复项和原始顺序。你要取值，得自己遍历找。

**另一个关键行为**：**`HTMLParser` 会自动把 `tag` 和属性名都转成小写**。你写 `<DIV CLASS="X">`，回调里拿到的是 `'div'` 和 `'class'`。这个设计是为了跟HTML“大小写不敏感”的规则对齐，但你如果依赖原始大小写，就要自己留一份原始数据。

### `handle_endtag(tag)`：结束标签

只给你标签名，没有属性。**注意：HTMLParser不检查开始标签和结束标签是否匹配**，它只负责“看到 `</xxx>` 就回调你”，配对逻辑得你自己在类里维护。

### `handle_data(data)`：标签之间的文本

`data` 是**两个标签之间的纯文本内容**。这里有一个和SAX完全一样的坑，后面单独说：**`data` 可能被拆成多次回调**。

### `handle_startendtag(tag, attrs)`：自闭合标签

专门处理 `<br/>`、`<img ... />` 这种自闭合写法。如果你不重写它，默认行为是**先调 `handle_starttag` 再调 `handle_endtag`**。你如果只关心“标签出现了”，不写它也行；想精确区分自闭合和普通标签，就重写它。

### 特殊字符：`handle_entityref` 和 `handle_charref`

原文特意提了一句：**特殊字符有两种，一种是英文表示的 `&nbsp;`，一种是数字表示的 `&#123;`，两种都能通过Parser解析出来**。

- `handle_entityref(name)`：`&nbsp;` 这种，`name` 是 `'nbsp'`；
- `handle_charref(name)`：`&#123;` 或 `&#x7b;` 这种，`name` 是 `'123'` 或 `'x7b'`。

**但有一个默认行为需要注意**：`HTMLParser` 默认 `convert_charrefs=True`，**大部分字符引用会被自动转换成对应的Unicode字符，直接塞进 `handle_data` 里**，不会触发 `handle_entityref`/`handle_charref`。只有 `script` 和 `style` 里的字符引用不会被转换。如果你需要拿到原始的 `&xxx;` 形式，创建解析器时要显式传 `convert_charrefs=False`。

---

## 坑一：`handle_data` 会被拆成多次，必须累加

这是HTMLParser最核心的坑，和上一页SAX的 `char_data` 一模一样。你有一段文本 `<p>Hello World</p>`，`handle_data` 理论上应该一次给你 `"Hello World"`，但**底层实现不保证**，它可能分两次给你 `"Hello"` 和 `" World"`，也可能在 `&amp;` 附近切开。

**正确写法**：用成员变量当缓冲区，`starttag` 清空、`handle_data` 累加、`endtag` 合并输出：

```python
class MyParser(HTMLParser):
    def __init__(self):
        super().__init__()
        self._buffer = ""                      # 文本缓冲区
        self._in_target = False                # 是否在目标标签内

    def handle_starttag(self, tag, attrs):
        if tag == "p":                         # 进入 <p> 标签
            self._in_target = True
            self._buffer = ""                  # 清空缓冲区

    def handle_data(self, data):
        if self._in_target:
            self._buffer += data               # 累加，不是赋值

    def handle_endtag(self, tag):
        if tag == "p" and self._in_target:     # 离开 <p> 标签
            print(self._buffer)                # 这里拿到的才是完整文本
            self._in_target = False
```

**不这么写，你的文本就会缺一块。**

---

## 坑二：它只做“词法解析”，不建树、不纠错

`HTMLParser` 的定位非常基础：**它只负责把HTML字符串切成“标签、文本、注释、实体”这些片段，然后回调你**。它**不构建DOM树，也不帮你修复不合法的HTML**。

官方文档明确说了：**这个解析器不检查结束标签是否与开始标签匹配**，也不会因为外层元素结束了就隐式关闭内层元素。更麻烦的是，**它有时候会直接跳过它不认识的标签**，导致你明明在源码里看到的元素，回调里根本拿不到。

**这意味着什么**：如果你要提取的是“某个 `div` 里所有 `a` 标签的 `href`”，你得自己在类里维护一个“当前在哪个标签里”的状态栈，靠 `starttag` 压栈、`endtag` 弹栈来实现。**HTMLParser不帮你做这件事**。

---

## 坑三：遇到畸形HTML，它可能直接停

真实世界的HTML并不总是规范的。`<x><y z=""o"" /></x>` 这种属性引号乱写的，`HTMLParser` 可能只识别了 `<x>` 就**静默停止解析**了，后面的内容全部丢失。

**如果你的目标页面是“野生HTML”（非你自己生成的），不要指望 `HTMLParser` 能兜住一切**。它的容错能力有限，遇到解析不出来的页面，要么先做一轮清洗，要么直接上 `lxml` 或 `html5lib` 这类更健壮的第三方解析器。

---

## 和上一页SAX的关系：同一个模型，不同的容错目标

| | **SAX（解析XML）** | **HTMLParser（解析HTML）** |
|---|---|---|
| **驱动方式** | 事件驱动，重写回调 | **完全一样** |
| **设计前提** | 假设输入是**格式良好的XML** | 假设输入是**语法松散的HTML** |
| **容错** | 遇到不合法XML直接报错 | **尽量容忍HTML的不规矩**，但仍有极限 |
| **数据完整性** | `char_data` 可能分段，需累加 | **同样如此**，`handle_data` 可能分段 |
| **建树** | 不建树 | 不建树 |

**你上一页学SAX学到的那套“start清空、data累加、end合并”的模式，在这一页原封不动地适用。** 唯一的区别是，HTMLParser多了一些专门对付HTML特性的回调（自闭合标签、注释、特殊字符），并且对“不严格语法”有更多的容忍。

---

## 练习的思路

原文的练习是：**拿一个网页的源码，提取会议的时间、名称、地点**。

思路不是“用HTMLParser直接找到那个元素”，而是：

1. **先肉眼看一下目标页面的源码结构**，确定时间、名称、地点分别被包在什么标签里，比如 `<time>...</time>`、`<h2>...</h2>`、`<address>...</address>`；
2. **在 `handle_starttag` 里判断“我进了目标标签”**，设置标志；
3. **`handle_data` 里如果标志为真，就把文本累加进缓冲区**；
4. **`handle_endtag` 里如果离开目标标签，就把缓冲区的内容存下来**，清空标志。

**HTMLParser不提供“按CSS选择器查找”的能力**，它只给你原始的事件流。你要提取结构化数据，状态机得自己写。

---

## 这页记住三件事

1. **HTMLParser是事件驱动的，继承它、重写 `handle_starttag` / `handle_endtag` / `handle_data`，然后 `feed()` 喂数据**。
2. **`handle_data` 会分段，必须累加，不能直接返回**；`attrs` 是元组列表，不是字典；标签名和属性名会自动转小写。
3. **它不建树、不纠错、不检查标签配对**，提取结构化数据得自己维护状态；野生HTML遇到它解不动，换 `lxml` 或 `html5lib`。

---

## 自测

1. HTMLParser和SAX在“工作模型”上是什么关系？它们唯一的关键区别是什么？
2. `handle_starttag` 的 `attrs` 是什么类型？为什么不是字典？标签名的大小写会被怎么处理？
3. 为什么 `handle_data` 必须累加，不能直接 `return data`？不累加会丢什么？
4. `convert_charrefs` 默认是什么值？它对你的 `handle_entityref` 和 `handle_charref` 有什么影响？
5. HTMLParser能帮你把 `<div><p>hello</div>` 这种标签不配对的HTML修好吗？为什么？