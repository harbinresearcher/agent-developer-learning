`argparse` 解决的是“命令行参数一多，手动解析 `sys.argv` 就崩”的问题。我们直接围绕它的**核心心智模型**和**实际编码中真正会踩的坑**来展开。

---

# argparse · 笔记

## 先聊痛点：为什么不用 `sys.argv` 手动切

`sys.argv` 是一个裸列表：`['copy.py', 'source.txt', 'copy.txt']`。简单场景下 `source = sys.argv[1]` 能用，但参数一旦带上 `--user`、`--port`、`-d` 这类开关，手动判断会变成一堆 `if` 和索引切片。更致命的是：**用户输错了，你没有任何提示；用户不知道有哪些参数，你也没有帮助信息**。

`argparse` 的核心思路是 **“声明式”** ：你告诉它“我这个程序接受哪些参数、每个参数长什么样”，剩下的解析、类型转换、校验、生成帮助信息，它全包了。

---

## 三件套：Parser → add_argument → parse_args

整个模块围绕一个对象和两个方法转。

### 第一步：创建解析器

```python
import argparse

parser = argparse.ArgumentParser(
    prog='backup',                          # 程序名，显示在帮助里
    description='Backup MySQL database.',   # 一行描述
)
```

`prog`、`description` 不写也行，但**写了之后 `--help` 的输出会专业很多**。不写的话，程序名默认取脚本文件名，描述为空。

### 第二步：用 `add_argument` 声明参数

参数分两类，这是最先要分清的东西。

**位置参数（positional）** ：直接写名字，**不写就报错**，没有默认值。

```python
parser.add_argument('outfile')              # 必须提供，位置由声明顺序决定
```

**可选参数（optional）** ：以 `-` 或 `--` 开头，**默认可以不提供**。

```python
parser.add_argument('--host', default='localhost')   # 可选，不写就用 localhost
```

但“可选”不等于“可以缺”。加上 `required=True`，可选参数就变成了**必须写，而且必须带 `--host` 这种关键字形式**，不能只写值。

```python
parser.add_argument('-u', '--user', required=True)   # 必须写，-u 或 --user 都行
```

`-u` 和 `--user` 是同一个参数的短名和长名，**短名给熟练用户省打字，长名给帮助文档和可读性**。

### 第三步：解析，拿到结果

```python
args = parser.parse_args()      # 解析 sys.argv
print(args.outfile, args.host)  # 通过属性访问，不是字典下标
```

`parse_args()` 返回的是一个 `Namespace` 对象，所有参数都是它的**属性**。你在 `add_argument` 里写的名字（去掉开头的 `--`）就是属性名。

---

## `add_argument` 里真正用得上的几个参数

| 参数 | 作用 | 一句话记住 |
|---|---|---|
| `type` | 把字符串转成指定类型 | `type=int`，传 `"3306"` 拿到的是 `3306` |
| `default` | 不提供时的默认值 | **和 `required=True` 互斥**，后者会忽略 `default` |
| `required` | 可选参数强制必须提供 | 位置参数不需要写，它天然必需 |
| `action='store_true'` | 开关标志，出现即 `True` | 不带值的参数，如 `--gzcompress` |
| `choices` | 限定取值范围 | 传了非法值直接报错，比自己在函数里判断更早 |
| `help` | 帮助信息 | **不写 `help`，`--help` 里这个参数就是空描述** |

### `type` 的边界

`type=int`、`type=float` 是最常用的。但注意：`type` 只适合**简单转换**，且转换函数只能抛出 `ValueError`、`TypeError` 或 `ArgumentTypeError` 才会被 `argparse` 捕获并友好地报错。像 JSON 解析、文件打开这类复杂逻辑，**不应该塞进 `type`**，应该在 `parse_args()` 之后自己做。

### `choices` 比 `type` 更适合“白名单”

如果要限定参数只能取几个值，用 `choices=['dev', 'staging', 'prod']`。传了不在列表里的值，`argparse` 会**在解析阶段直接拒绝**，错误信息里会列出所有合法选项，比你自己在业务代码里判断要清楚得多。

---

## 几个原文没展开、但实际编码绕不开的坑

### 坑一：`required=True` 和 `default` 不要同时写

对于一个被标记为 `required=True` 的可选参数，**`default` 是无效的**，因为它永远不会走到“不提供”的分支。写 `required=True, default='xxx'` 是自相矛盾的，要么让它必须提供，要么给默认值让它可选。

### 坑二：位置参数和可选参数混用时，顺序有讲究

`parse_args()` 能处理穿插，但**位置参数的数量和位置是固定的**。如果你声明了 `outfile` 一个位置参数，用户输入 `backup.sql --host localhost` 和 `--host localhost backup.sql` 通常都能解析，但一旦有多个位置参数，它们的**声明顺序就是命令行里的先后顺序**，不要指望用户按你心里的顺序输入。

### 坑三：负数参数会被误判为选项

如果某个参数的值可能是负数（比如 `--threshold -0.5`），`argparse` 默认会把 `-0.5` 当作一个“选项”而不是值。**Python 3 在解析器里没有看起来像负数的选项时，会允许 `-` 开头的值作为位置参数**，但可选参数的值仍然可能被吞。安全的写法是 `--threshold=-0.5`（用等号连起来），或者用 `type=float` 配合明确的短选项。

### 坑四：子命令（subparsers）是另一个层级的东西

当你的工具像 `git` 一样有 `commit`、`push` 这样的子命令时，不要用一堆 `if args.command == 'commit'` 硬写。`add_subparsers()` 可以创建**独立的子解析器**，每个子命令有自己的参数集，而且可以把 `set_defaults(func=...)` 绑定到对应的处理函数上，解析完直接 `args.func(args)` 分发。

```python
subparsers = parser.add_subparsers(dest='command', required=True)

train = subparsers.add_parser('train')
train.add_argument('--model', required=True)

evaluate = subparsers.add_parser('evaluate')
evaluate.add_argument('--model', required=True)
```

这是 `argparse` 里**唯一值得在入门阶段就了解的结构性功能**，因为一旦参数超过 5 个，不分层就会乱。

---

## 自测

1. 位置参数和可选参数的核心区别是什么？`required=True` 用在哪个上面才有意义？
2. `type=int` 和 `choices=[...]` 分别解决什么类型的问题？为什么说 `choices` 是“白名单”而 `type` 是“转换”？
3. 为什么 `required=True` 和 `default='xxx'` 不应该同时写在同一个可选参数上？
4. 用户输入 `--threshold -0.5` 时可能出什么问题？怎么绕过？
5. `parse_args()` 返回的是什么类型的对象？参数名和属性名之间是什么对应关系？
6. 子命令机制适合什么场景？不用 `add_subparsers`，你会怎么写，代价是什么？