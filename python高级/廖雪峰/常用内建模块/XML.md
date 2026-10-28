这页原文的主线其实特别清楚：**XML解析有两条路——DOM和SAX，廖雪峰推荐SAX，然后用SAX把一套事件回调走一遍，最后说生成XML直接拼字符串就完事**。咱们就顺着这个骨架，把“为什么SAX、怎么用SAX、SAX哪里会坑你”讲透。

---

# XML · 笔记

## 先聊痛点：XML是一棵“有嵌套结构的树”，你怎么把它变成数据？

JSON你直接用 `json.loads()` 就拿到dict了，XML不行。XML长这样：

```xml
<bookstore>
  <book category="COOKING">
    <title lang="en">Everyday Italian</title>
  </book>
</bookstore>
```

这是一个**树**：`bookstore` 是根，`book` 是它的孩子，`title` 是 `book` 的孩子，`category` 和 `lang` 是贴在标签上的属性。你要做的不是“读文本”，是**遍历这棵树，把节点、属性、文本内容按你的需要提取出来**。

Python提供两条路，区别在于 **“什么时候把什么放进内存”**。

---

## 两条路线：DOM vs SAX，选错就是内存和代码量的交换

原文对两者的定性非常干脆，我直接把结论钉在这里：

| | **DOM** | **SAX** |
|---|---|---|
| **内存** | **整个XML一次性读进内存**，解析成树 | **边读边解析，不存整棵树** |
| **速度** | **慢** | **快** |
| **代码** | 拿到树之后随便遍历，**简单** | **必须自己写事件回调**，麻烦 |
| **适合** | XML很小，你想反复翻来覆去查 | **XML可能很大，你只关心其中一部分节点** |

廖雪峰的原话是：**正常情况下，优先考虑SAX，因为DOM实在太占内存**。一个几MB的XML，DOM读进来再建树，内存占用可能翻好几倍；SAX从头到尾扫一遍，内存只跟“你当前处理到哪个节点”有关，跟文件大小基本无关。

但代价也很直接：**DOM给你一棵现成的树，你想怎么爬怎么爬；SAX不给你树，它只在“碰到一个开始标签、碰到一段文本、碰到一个结束标签”的时候通知你，你得自己把这些通知拼成你需要的数据**。

---

## SAX怎么工作：三个事件，一条流水线

SAX是**事件驱动**的。你把一个XML扔给它，它从头扫到尾，扫到不同的位置就**回调你写的函数**。你真正需要关心的只有三个事件：

拿 `<a href="/">python</a>` 举例，SAX会依次产生：

1. **`start_element`**：读到 `<a>` 时触发，**给你标签名和属性**；
2. **`char_data`**：读到 `python` 这段文本时触发，**给你文本内容**；
3. **`end_element`**：读到 `</a>` 时触发，**给你标签名**。

**一个元素的完整生命周期，就是这三件事按顺序发生。** 嵌套元素会递归地触发，比如 `<book><title>X</title></book>`，顺序是 `start book` → `start title` → `char X` → `end title` → `end book`。

---

## 最小SAX实战：把事件打印出来

原文用的是 `xml.parsers.expat` 里的 `ParserCreate`，**不继承任何类，纯手动挂三个函数**：

```python
from xml.parsers.expat import ParserCreate     # expat 是底层解析器，ParserCreate 创建它

class DefaultSaxHandler(object):                # 一个普通的类，不需要继承 ContentHandler
    def start_element(self, name, attrs):        # name 是标签名，attrs 是属性字典
        print('sax:start_element: %s, attrs: %s' % (name, str(attrs)))

    def end_element(self, name):                 # 读到闭合标签时触发
        print('sax:end_element: %s' % name)

    def char_data(self, text):                   # 读到文本节点时触发
        print('sax:char_data: %s' % text)

xml = r'''<?xml version="1.0"?>
<ol>
    <li><a href="/python">Python</a></li>
    <li><a href="/ruby">Ruby</a></li>
</ol>
'''

handler = DefaultSaxHandler()                    # 创建 handler 实例
parser = ParserCreate()                          # 创建解析器
parser.StartElementHandler = handler.start_element   # 把三个函数挂到解析器上
parser.EndElementHandler = handler.end_element
parser.CharacterDataHandler = handler.char_data
parser.Parse(xml)                                # 开始解析，回调会在这行里全部触发
```

**你要记住的不是 expat 的细节，而是这个结构：一个 handler 对象 + 三个函数 + 挂到 parser 上 + `Parse()` 启动。** 后面不管你用 expat 还是 `xml.sax`，骨架都是这个。

---

## 第一个坑，也是最核心的坑：`char_data` 会被拆成多次

这是SAX里最容易写出错误数据的点，原文专门提了一句：**读取一大段字符串时，`CharacterDataHandler` 可能被多次调用，所以需要自己保存起来，在 `EndElementHandler` 里面再合并**。

什么意思？你有一个 `<title>Everyday Italian</title>`，`char_data` 理论上应该一次性给你 `"Everyday Italian"`。但**底层解析器不保证这一点**，它可能分两次给你 `"Everyday"` 和 `" Italian"`，也可能分三次。**你如果在 `char_data` 里直接 `return text`，拿到的就是碎片，不是完整文本。**

**正确写法**：在 `start_element` 里把“当前文本缓冲区”清空，`char_data` 里**累加**，`end_element` 里**合并并决定要不要保存**：

```python
class Handler:
    def __init__(self):
        self.current_text = ""                   # 文本缓冲区
        self.current_tag = ""                    # 当前在哪个标签里

    def start_element(self, name, attrs):
        self.current_tag = name                  # 记住当前标签名
        self.current_text = ""                   # 进入新标签，清空缓冲区

    def char_data(self, text):
        self.current_text += text                # 累加，不是覆盖

    def end_element(self, name):
        if name == "title":
            print("完整的文本是：", self.current_text)   # 到这里才是完整内容
```

**记住这个模式：`start` 清空、`char_data` 累加、`end` 合并。** 不这么写，你的文本就会缺一块。

---

## 第二个坑：你不需要处理每一个事件，只处理你关心的

SAX会把**整个文档的每一个节点**都回调给你，包括你不关心的那些。如果你在 `start_element` 里对每个标签都做复杂操作，代码会又乱又慢。

**正确思路**：在 `start_element` 里判断 `name`，只对目标标签设置标志或开始收集数据；`end_element` 里同样判断 `name`，只处理目标标签的收尾。**其他标签的事件，直接 `pass` 掉**。这就是原文小结说的：**“注意找出自己感兴趣的节点，响应事件时，把节点数据保存起来”**。

---

## 生成XML：廖雪峰的结论是“直接拼字符串”，但有一个前提

原文对生成XML的态度非常实用主义：**99%的情况下需要生成的XML结构都非常简单，最简单也是最有效的方法是拼接字符串**。

```python
L = []
L.append(r'<?xml version="1.0"?>')               # XML声明
L.append(r'<root>')                              # 开始标签
L.append(encode('some & data'))                  # 文本内容，注意 & 要转义
L.append(r'</root>')                             # 结束标签
return ''.join(L)                                # 拼成一个完整字符串
```

**但拼接字符串有一个必须记住的坑：特殊字符要转义。** XML里的 `&`、`<`、`>` 不能直接出现在文本内容里，`&` 必须写成 `&amp;`，`<` 写成 `&lt;`，`>` 写成 `&gt;`。上面代码里 `encode()` 就是干这个的。**不转义，生成的XML直接是非法格式。**

**那什么时候不该拼字符串？** 原文说得很干脆：**“如果要生成复杂的XML，建议你不要用XML，改成JSON”**。这句话背后的意思是：XML的复杂度不值得你用字符串拼接去硬扛，如果结构复杂到拼接维护不了，换JSON是更理性的选择。

---

## 补充一个原文没提、但你实际会碰到的选项：ElementTree

廖雪峰这页只讲了SAX和DOM，但**Python标准库里还有一个 `xml.etree.ElementTree`，它在实际项目里的使用频率远高于SAX和DOM**。它的定位介于两者之间：**像DOM一样给你一棵树，可以按标签名查找，但底层做了流式优化，内存占用比DOM小，API也比SAX那套回调干净得多**。

```python
import xml.etree.ElementTree as ET

tree = ET.parse('data.xml')                      # 解析文件，返回 ElementTree
root = tree.getroot()                            # 拿到根节点
for book in root.findall('book'):                # 按标签名查找子节点
    title = book.find('title').text              # 取文本
```

**这页学SAX是为了理解“事件驱动解析XML”这个模型**，但你以后真正写业务代码，**优先看ElementTree**。SAX留给“XML巨大、内存扛不住、只需要扫一遍提取部分数据”的场景。

---

## 这页记住三件事

1. **DOM把整棵树读进内存，SAX边读边回调**；XML大、只关心部分节点，选SAX。
2. **SAX的三个核心事件是 `start_element`、`char_data`、`end_element`**；`char_data` 必须累加、在 `end_element` 里合并，否则文本会碎。
3. **生成简单XML直接拼字符串，注意转义 `&`、`<`、`>`**；结构复杂就换JSON，别跟XML硬耗。实际写业务代码，**先看 `xml.etree.ElementTree`**。

---

## 自测

1. DOM和SAX在“内存”和“代码复杂度”上分别做了什么交换？什么场景下SAX的优势会压过它的麻烦？
2. SAX里一个 `<a href="/">python</a>` 会依次触发哪三个事件？每个事件回调收到什么参数？
3. 为什么 `char_data` 可能被多次调用？不累加、直接返回，会出什么结果？
4. 生成XML时，文本里的 `&` 不转义会怎样？原文为什么说“复杂XML不如换JSON”？
5. 你手头有一个500MB的XML日志文件，只想提取其中所有 `<error>` 标签的文本，用SAX、DOM还是ElementTree？为什么？