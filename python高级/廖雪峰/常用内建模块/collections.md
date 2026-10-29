> **重要程度：高**｜**手练：需要**
> 重点：Counter、defaultdict、deque、namedtuple。

collections 提供多种集合工具。本页介绍其中六种。

---

# collections · 笔记

## 先聊：为什么要有这个模块

Python内置的tuple、list、dict用起来方便，但都有明显的局限：tuple用下标访问，读代码的人不知道`(1, 2)`是什么意思；list头部插入删除效率极低；dict的key不存在直接报错，有时候你想要一个默认值；dict在Python 3.7之前不保证顺序。collections就是**提供多个专门的集合工具，本页介绍六种**。你不需要全部记住，但看到某个场景，应该能条件反射地想到它。

---

## namedtuple：让tuple的每个位置都有名字

**痛点**：`p = (1, 2)`表示一个坐标，但`p[0]`和`p[1]`到底哪个是x、哪个是y，光看代码看不出来。专门定义一个class又太小题大做。

**解法**：用`namedtuple`创建一个“带属性名”的tuple。

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])   # 创建一个Point类型，有两个字段x和y
p = Point(1, 2)                           # 像普通tuple一样创建实例
print(p.x, p.y)                           # 1 2，现在可以用属性名访问
```

`namedtuple`是一个函数，第一个参数是类型名，第二个参数是字段名列表。创建出来的`Point`是tuple的子类，所以它依然有tuple的全部行为（不可变、可迭代、支持下标），但多了一套属性访问方式。

**适用场景**：任何“固定几个字段、但用下标访问可读性差”的数据，比如坐标、数据库查询的一行结果、配置项。

---

## deque：头部操作效率高的双向列表

**痛点**：list按索引访问很快，但**头部插入和删除很慢**，因为list是线性存储，在头部插一个元素，后面所有元素都要往后挪一位，数据量一大就废了。

**解法**：`deque`（double-ended queue，双端队列）是两头都高效的列表。

```python
from collections import deque

q = deque(['a', 'b', 'c'])   # 用列表初始化一个deque
q.append('x')                # 尾部追加，和list一样
q.appendleft('y')            # 头部插入，这是list做不到的高效操作
print(q)                     # deque(['y', 'a', 'b', 'c', 'x'])

q.pop()                      # 尾部弹出
q.popleft()                  # 头部弹出，同样高效
```

`deque`除了支持list的`append`/`pop`，还多了`appendleft`/`popleft`，专门用于高效操作头部。**适用场景**：队列（先进先出）和栈（后进先出）用`deque`比用list更合适，尤其是需要频繁在头部操作时。

---

## defaultdict：key不存在时给你一个默认值

**痛点**：用普通dict时，访问一个不存在的key会直接抛`KeyError`。你每次都得先`if key in d`判断一下，或者用`d.get(key, default)`，写起来啰嗦。

**解法**：`defaultdict`在key不存在时，自动调用你传入的函数生成一个默认值，而不是报错。

```python
from collections import defaultdict

dd = defaultdict(lambda: 'N/A')   # 默认值由这个函数生成
dd['key1'] = 'abc'
print(dd['key1'])                 # 'abc'，key存在，正常返回
print(dd['key2'])                 # 'N/A'，key不存在，返回默认值，不报错
```

**关键点**：默认值是**调用函数返回的**，这个函数在你创建`defaultdict`时就传进去了。缺失键的方括号访问会调用工厂并插入新值；`get()` 不会触发默认工厂。

**最常用的写法**是`defaultdict(list)`或`defaultdict(int)`，分别用于“把值收集成列表”和“计数”的场景。

---

## OrderedDict：记住key的插入顺序

**痛点**：普通dict的key在Python 3.7之前不保证顺序，迭代时顺序是随机的。即使现在3.7+的dict已经保证插入顺序，OrderedDict依然有它的特殊用途（见下文）。

**解法**：`OrderedDict`的key**严格按照插入顺序排列**。

```python
from collections import OrderedDict

od = OrderedDict()
od['z'] = 1
od['y'] = 2
od['x'] = 3
print(list(od.keys()))   # ['z', 'y', 'x']，按插入顺序，不是字母排序
```

注意：**是按插入顺序，不是按key本身排序**。先插`z`再插`y`，`z`就在前面。

**OrderedDict真正独特的地方**：它可以实现一个**FIFO（先进先出）的dict**，需要自己编写容量和淘汰逻辑，才能在容量超出限制时删除最早添加的 key。原文的`LastUpdatedOrderedDict`示例就是做这件事：当容量满了，用`popitem(last=False)`弹出最早插入的那个key。这是普通dict做不到的。

**适用场景**：需要控制顺序的缓存、需要按插入顺序序列化的配置、以及需要FIFO淘汰策略的场景。

---

## ChainMap：把多个dict串成一个逻辑上的dict

**痛点**：你有一堆来源不同的配置——命令行参数、环境变量、默认值——你想按优先级顺序查找，先查命令行，没有再查环境变量，再没有用默认值。如果每个来源单独判断，代码会变成一堆嵌套的`if`。

**解法**：`ChainMap`把多个dict串起来，查找时**按顺序在内部依次搜索**。

```python
from collections import ChainMap

defaults = {'color': 'red', 'user': 'guest'}
command_line_args = {'user': 'bob'}   # 假设命令行传了-u bob
env_vars = {'color': 'green'}         # 假设环境变量设了color=green

combined = ChainMap(command_line_args, env_vars, defaults)   # 优先级：命令行 > 环境变量 > 默认值
print(combined['color'])   # 'green'，命令行没有color，查到环境变量
print(combined['user'])    # 'bob'，命令行有user，直接用
```

`ChainMap` 是映射对象，不是 dict 子类，但**查找时按照构造时传入的顺序，依次在内部dict中搜索**，找到就返回，不再继续往后找。**适用场景**：多级配置系统、参数优先级查找。

---

## Counter：计数器，专门用来数数

**痛点**：统计一个字符串里每个字符出现的次数，手动写`for`循环加dict计数，代码又长又容易写错。

**解法**：`Counter`是dict的子类，专门用来计数。

```python
from collections import Counter

c = Counter('programming')   # 直接传入可迭代对象，自动统计每个元素出现次数
print(c)                     # Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, ...})

c.update('hello')            # 也可以一次性把另一批数据加进来计数
print(c)                     # 各字符的计数被更新
```

`Counter`的用法非常直接：**传入任何可迭代对象，它自动帮你数每个元素出现了几次**，返回的是一个dict子类，key是元素，value是计数。**适用场景**：词频统计、字符统计、任何“每个东西出现多少次”的场景。

---

## 排雷区

**1. `defaultdict`的默认值函数只接受无参数的函数**。`lambda: 'N/A'`可以，但`lambda x: x`不行，因为`defaultdict`调用它时不传任何参数。

**2. `OrderedDict`在Python 3.7+的“顺序”优势已经被普通dict追上了**，普通dict现在也保证插入顺序。但`OrderedDict`的`popitem(last=False)`和`move_to_end()`这些**独有方法**，普通dict没有，FIFO淘汰这类需求依然要用它。

**3. `ChainMap`的写入只作用于第一个dict**。`combined['new_key'] = 'x'`会把新key写进**第一个**内部dict，而不是所有dict。读是链式查找，写不是。

**4. `Counter`的`update`是“累加”不是“替换”**。`c.update('hello')`会把`'h'`、`'e'`、`'l'`、`'o'`的计数**加到已有值上**，不是覆盖。

---

## 自测

1. `namedtuple`创建的对象，`isinstance(p, tuple)`返回什么？为什么它既有tuple的不变性，又能用属性访问？
2. list头部插入为什么慢？`deque`为什么快？`appendleft`和`popleft`分别对应什么操作？
3. `defaultdict`和普通dict在key不存在时的行为差在哪？默认值是怎么产生的？
4. `OrderedDict`的key按什么顺序排列？它在Python 3.7+还有什么普通dict替代不了的能力？
5. `ChainMap(command_line, env, defaults)`查找`'color'`时，搜索顺序是什么？找到第一个就停还是继续找？
6. `Counter('programming')`返回的是什么类型？`c.update('hello')`之后，`'g'`的计数会变吗？为什么？
手练：用 `Counter` 统计词频，用 `defaultdict(list)` 分组，再用 `deque` 写一个先进先出队列；确认读取缺失键时 `get()` 和方括号的区别。