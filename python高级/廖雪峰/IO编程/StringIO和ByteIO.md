> **重要程度：中**｜**手练：建议**
> 重点：字符串和字节流、seek、上下文管理。

# StringIO 和 BytesIO · 笔记

## 先搞清楚：这俩是干嘛的

前面读文件、写文件，数据最终落在**磁盘**上。但很多时候你只是想**在内存里临时拼一段数据、或者把一段字符串当文件来读**，不想真的创建文件。`StringIO` 和 `BytesIO` 就是干这个的：**它们在内存里模拟出一个文件对象，支持许多文件式接口，但并非所有文件操作都适用**，`read`、`write`、`readline`、`seek` 全都有。

区别就一条：**`StringIO` 操作 `str`，`BytesIO` 操作 `bytes`**。混用直接 `TypeError`。

---

## StringIO：内存里的“文本文件”

### 写入：像文件一样写，但用 `getvalue()` 拿内容

```python
from io import StringIO                 # 从 io 模块导入 StringIO

f = StringIO()                          # 在内存里创建一个空的文本流
f.write('hello')                        # 写入字符串，返回写入的字符数 5
f.write(' ')                            # 再写一个空格，返回 1
f.write('world!')                       # 再写 world!，返回 6
print(f.getvalue())                     # getvalue() 取出全部内容：'hello world!'
```

`write()` 的返回值是**写入的字符数**，不是内容本身。想拿到写进去的东西，用 `getvalue()`。

**关键坑**：写完立刻 `read()` 是读不到东西的，因为**读写指针停在末尾**。要读，先 `seek(0)` 把指针拨回开头。

```python
f = StringIO()
f.write('hello')
print(f.read())                         # ''，指针在末尾，读不到
f.seek(0)                               # 把指针拨回开头
print(f.read())                         # 'hello'
```

### 读取：用一个字符串初始化，然后像文件一样读

```python
from io import StringIO

f = StringIO('Hello!\nHi!\nGoodbye!')   # 直接用现成字符串初始化，指针在开头
while True:
    s = f.readline()                    # 逐行读
    if s == '':                         # 读完了，readline 返回空字符串
        break
    print(s.strip())                    # strip() 去掉行尾换行符
```

输出三行：`Hello!`、`Hi!`、`Goodbye!`。用字符串初始化时，**指针在开头**，可以直接 `readline()` 或 `read()`。

---

## BytesIO：内存里的“二进制文件”

文本用 `StringIO`，**二进制数据（图片、音视频、任意字节）用 `BytesIO`**。

```python
from io import BytesIO

f = BytesIO()                           # 创建一个空的二进制流
f.write('中文'.encode('utf-8'))          # 把字符串编码成 bytes 再写，返回 6（字节数）
print(f.getvalue())                     # b'\xe4\xb8\xad\xe6\x96\x87'
```

注意 `write` 的是 **`bytes`**，不是 `str`。“中文”两个字 UTF-8 编码后是 6 个字节。

用现成的 `bytes` 初始化，同样可以直接读：

```python
f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')   # 用 bytes 初始化，指针在开头
print(f.read())                              # b'\xe4\xb8\xad\xe6\x96\x87'
```

和 `StringIO` 一样，**写完之后要读，先 `seek(0)`**。

---

## `getvalue()` 和 `read()` 的根本区别

这是这页最容易糊的地方，单独拎出来。

- **`getvalue()`**：返回缓冲区里的**全部内容**，**不受指针位置影响**；
- **`read()`**：从**当前指针位置**开始读，读完指针移到末尾。

```python
f = StringIO('1\n2\n3')
print(f.getvalue())     # '1\n2\n3'，全部内容，指针位置无所谓
print(f.read())         # '1\n2\n3'，从当前位置（开头）读到尾
print(f.read())         # ''，指针已经在末尾了，没东西可读
print(f.getvalue())     # '1\n2\n3'，getvalue 依然拿得到全部
```

**记住：`getvalue()` 是“看全部”，`read()` 是“从指针处读”**。要反复读同一段内容，用 `getvalue()` 最省事，或者每次读之前 `seek(0)`。

---

## 这东西拿来干嘛

核心价值就一句：**让不需要落盘的数据，也能用文件接口来处理**。

常见的场景：用 `csv.writer` 往 `StringIO` 里写 CSV 内容，不建临时文件；把某个函数的 `print` 输出重定向到 `StringIO` 里再解析；测试的时候用 `BytesIO` 模拟文件输入，不用真的准备测试文件。凡是“需要一个文件对象，但我不想真的创建文件”的地方，就是它俩出场的时候。

---

## 自测

1. `StringIO` 和 `BytesIO` 分别操作什么类型的数据？混用会怎样？
2. 往 `StringIO` 里 `write` 之后直接 `read()`，为什么读不到？怎么解决？
3. `getvalue()` 和 `read()` 的关键区别是什么？
4. 用 `StringIO('...')` 初始化时，指针在开头还是末尾？和空 `StringIO()` 写完之后的状态有什么不同？
5. `BytesIO` 的 `write` 接受 `str` 还是 `bytes`？“中文”两个字 UTF-8 编码后是几个字节？

## 工作中注意

内存流的数据占用进程内存；大文件不一定适合完整放入 `BytesIO`。部分依赖要求真实文件描述符，此时内存流不能直接替代磁盘文件。用完关闭，或用上下文管理器管理生命周期。