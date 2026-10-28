这页原文其实在讲一件事：**把 with 语句背后那套繁琐的协议，用更省事的方式实现出来**。从最原始的 `__enter__`/`__exit__`，到 `@contextmanager` 用生成器简化，再到 `@closing` 把“没有上下文管理的对象”硬包装成能用的。咱们顺着“痛点 → 手动实现 → 两次简化”这条线走。

---

# contextlib · 笔记

## 先聊痛点：with 省了你的手，但没省写类的心

你每天都在写 `with open(...) as f:`，知道它不用手动 `close()`，出了事也自动关。但有没有想过：**为什么 `open()` 能用在 `with` 里，而你自己写的类不能？**

答案是：**`with` 不认“文件”，它只认一套协议**。任何对象，只要实现了 `__enter__` 和 `__exit__` 这两个方法，就能用在 `with` 语句里。文件对象实现了，所以能用；你随便写个 `class Foo: pass`，没实现，直接用 `with` 就报 `AttributeError`。

所以想让自己的资源对象支持 `with`，你得手写这两个方法。这就是第一层麻烦。

---

## 第一层：手写 `__enter__` 和 `__exit__`

原文给的例子：

```python
class Query:
    def __init__(self, name):
        self.name = name

    def __enter__(self):                # 进入 with 块时自动调用
        print('Begin')
        return self                      # 返回值绑定到 as 后面的变量

    def __exit__(self, exc_type, exc_value, traceback):   # 离开 with 块时自动调用
        if exc_type:                     # 有异常，exc_type 不为 None
            print('Error')
        else:
            print('End')
        return False                     # 返回 False 表示异常继续往外抛

    def query(self):
        print(f'Query info about {self.name}...')
```

用起来：

```python
with Query('Bob') as q:      # __enter__ 被调，返回值赋给 q
    q.query()                 # 块内正常干活
# 离开 with 块，__exit__ 被调，打印 End
```

**三个关键行为要钉死**：

1. **`__enter__` 的返回值，就是 `as` 后面那个变量**。`return self` 是最常见的写法，这样 `q` 拿到的是对象本身，能继续调 `query()`。
2. **`__exit__` 的四个参数**：`exc_type`、`exc_value`、`traceback`，正常结束时三个都是 `None`；块里抛了异常，它们就是异常的类型、实例和回溯信息。
3. **`__exit__` 返回 `False` 或 `None`，异常继续往外抛；返回 `True`，异常被吞掉**。不写 `return` 默认就是 `None`，等于 `False`。

到这儿你能用了，但**每做一个支持 `with` 的类，就得写一遍 `__enter__` 和 `__exit__`**，代码里全是模板。所以有了第二层简化。

---

## 第二层：`@contextmanager`，用生成器替代两个方法

这是 `contextlib` 最核心的东西。思路是：**你不需要写一个类，只需要写一个生成器函数，在 `yield` 之前放 `__enter__` 该干的事，`yield` 后面放 `__exit__` 该干的事**。

```python
from contextlib import contextmanager

@contextmanager
def create_query(name):
    print('Begin')                  # yield 之前 = __enter__ 的逻辑
    q = Query(name)
    yield q                          # yield 出去的值，就是 as 后面拿到的变量
    print('End')                     # yield 之后 = __exit__ 的逻辑

with create_query('Bob') as q:       # q 拿到的是 yield 出来的 Query 实例
    q.query()
```

执行顺序是固定的：**进入 `with` 先跑到 `yield`，`yield` 的值赋给 `q`；然后执行 `with` 块内的代码；块结束后，从 `yield` 下一行继续跑，直到函数结束**。

你不需要写 `__enter__` 和 `__exit__` 两个方法，只需要一个函数，`yield` 就是那条分界线。**这是 `@contextmanager` 的全部魔法**。

### 它的通用形态：前后自动执行

原文给的 `tag` 例子最能说明问题：

```python
@contextmanager
def tag(name):
    print(f"<{name}>")               # with 块之前执行
    yield                             # 不产出值，as 后面不接变量
    print(f"</{name}>")               # with 块之后执行

with tag("h1"):
    print("hello")
    print("world")
```

输出：

```
<h1>
hello
world
</h1>
```

**`yield` 之前是“前置动作”，`yield` 之后是“后置动作”，`with` 块内的代码夹在中间**。这个模式可以用来做计时、打日志、加锁、事务——任何“前后各做一件事”的场景。

---

## 第三层：`@closing`，把没有 `__enter__`/`__exit__` 的对象硬包装成 with 可用

问题来了：**有些对象有 `close()` 方法，但没实现 `__enter__`/`__exit__`**，比如 `urllib.request.urlopen()` 返回的对象。它明明需要关闭，但直接 `with urlopen(...) as page:` 是用不了的。

`closing()` 就是干这个的：**接收一个有 `close()` 方法的对象，把它包装成一个上下文管理器**，离开 `with` 块时自动调 `close()`。

```python
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('https://www.python.org')) as page:
    for line in page:
        print(line)
# 离开 with 块，page.close() 自动被调
```

它的内部实现极简，本质就是一个 `@contextmanager` 包了一层 `try...finally`：

```python
@contextmanager
def closing(thing):
    try:
        yield thing                  # 把对象交给 with 块使用
    finally:
        thing.close()                # 不管块内是否异常，一定关
```

**`closing` 只适用于有 `close()` 方法的对象**，它不负责 `__enter__` 里可能需要的初始化逻辑。有 `close()` 但没上下文协议，用它；没 `close()`，它救不了。

---

## 选型：什么时候用哪个

| 你的情况 | 用什么 |
|---|---|
| 自己写类，想支持 `with` | 手写 `__enter__` / `__exit__` |
| 只需要“前后各做一件事”，不需要维护复杂状态 | `@contextmanager` |
| 手头有个对象，有 `close()` 但没上下文协议 | `closing()` |

**日常写代码，`@contextmanager` 是首选**，因为它用生成器代替了一整个类，代码量少、可读性高。只有在你需要**在 `with` 块结束后决定“异常要不要吞掉”**时，才需要回到手写 `__exit__`，因为 `@contextmanager` 默认不吞异常。

---

## 几个容易踩的坑

**1. `@contextmanager` 里 `yield` 必须恰好一次。** 生成器不 `yield`，`with` 块拿不到值；`yield` 两次，`with` 块结束后会继续往下跑，行为错乱。

**2. `@contextmanager` 默认不吞异常。** `with` 块里抛了异常，`yield` 那一行会收到异常，如果你不 `try...finally` 兜住，`yield` 后面的清理代码**可能不会执行**。要保证清理一定跑，把 `yield` 包在 `try...finally` 里：

```python
@contextmanager
def safe():
    try:
        yield
    finally:
        print("cleanup")      # 不管块内是否异常，一定执行
```

**3. `closing` 不负责“进入时初始化”。** 它只保证“退出时关闭”。如果你需要“打开时做点什么”，`closing` 不够，得自己写 `@contextmanager`。

**4. 手写 `__exit__` 时，`return True` 会吞掉异常，但吞异常通常是坏习惯。** 除非你明确知道“这个异常应该被忽略”，否则不写 `return` 或 `return False`，让异常正常往上抛。

---

## 自测

1. 一个对象想用在 `with` 语句里，必须实现哪两个方法？`__enter__` 的返回值绑定到哪里？
2. `__exit__` 的三个参数分别是什么？`return True` 和 `return False` 对异常的影响有什么区别？
3. `@contextmanager` 里 `yield` 之前、`yield` 本身、`yield` 之后，分别对应 `with` 语句的哪一部分？
4. `@contextmanager` 默认会吞异常吗？想让清理代码一定执行，该怎么写？
5. `closing()` 能用在什么样的对象上？它内部靠什么保证退出时一定关闭？
6. 你写了一个类，有 `close()` 方法但没有 `__enter__`/`__exit__`，想让它支持 `with`，三种方案里哪种最省事？