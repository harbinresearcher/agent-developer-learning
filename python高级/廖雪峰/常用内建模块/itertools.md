这页原文其实不复杂，但藏了一个非常关键的定性：**itertools 返回的全是迭代器，不是 list，只有 for 循环迭代时才算数**。这句话是整页的地基，先立住它，剩下的函数都是这个原则的应用。

---

# itertools · 笔记

## 先钉死一句话：它返回的是“流水线”，不是“仓库”

你平时 `range(10)`、`[x for x in ...]` 拿到的是一车货，已经装好了。itertools 的函数返回的是**一条传送带**，你不动它，它什么都不生产；你开始迭代，它才一个一个吐出来。原文在“小结”里专门强调了这一点：**返回值不是 list，而是 Iterator，只有用 for 循环迭代的时候才真正计算**。

这个定性决定了它的价值：**你不需要在内存里装下所有数据，只需要“按需生产”**。所以它能处理无限序列，也能用极低的内存跑大数据。

---

## 无限迭代器：三个函数，默认“停不下来”

原文先给了三个能无限产出元素的函数，这是 itertools 最能体现“惰性”的地方。

**`count(start, step)`**：从 `start` 开始，每次加 `step`，默认从 0 开始、步长为 1。

```python
natuals = itertools.count(1)     # 1, 2, 3, 4, ... 无限
```

**`cycle(iterable)`**：把一个序列无限重复下去。

```python
cs = itertools.cycle('ABC')      # A, B, C, A, B, C, ... 无限
```

**`repeat(elem, times)`**：把一个元素重复 `times` 次，**不传 `times` 就无限重复**。

```python
ns = itertools.repeat('A', 3)    # A, A, A，传了 3 就只重复 3 次
```

**关键认知**：这三个函数**创建的时候并不会在内存里生成无限个元素**，它们只是“准备好了生产规则”，你迭代一次，它生产一个。所以创建 `count(1)` 是瞬间完成的，不会卡死。

---

## `takewhile`：给无限序列“截一刀”

无限迭代器本身没法直接用，你总得让它停下来。原文给的方案是 **`takewhile(predicate, iterable)`**：**只要条件为真就继续取，条件第一次为假就立刻停**。

```python
natuals = itertools.count(1)                              # 1, 2, 3, ...
ns = itertools.takewhile(lambda x: x <= 10, natuals)      # 取到 10 就停
list(ns)                                                   # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

注意 `takewhile` 和 `filter` 的区别：**`filter` 是“满足条件的留下，不满足的跳过，继续往后找”；`takewhile` 是“一旦遇到第一个不满足的，后面的全部不要了，直接停”**。处理无限序列只能靠 `takewhile` 或 `islice` 这类截断函数，因为无限序列没有“后面”可以跳。

---

## `chain`：把多个可迭代对象串成一条传送带

**`chain(*iterables)`** 做的事就是**首尾相接**：把传入的所有可迭代对象依次串联，形成一个更大的迭代器。

```python
for c in itertools.chain('ABC', 'XYZ'):
    print(c)                     # A B C X Y Z
```

它不要求传入的都是同一种类型，list、tuple、字符串、生成器都可以混着串。**等价于你写两层嵌套 for 循环逐个 yield**，但 `chain` 是 C 层面实现的，更快、更省事。

---

## `groupby`：按 key 分组，但有一个致命的“相邻”限制

**`groupby(iterable, keyfunc)`** 把**相邻的、key 相同的元素**挑出来放在一组，返回 `(key, group_iterator)` 的序列。

```python
for key, group in itertools.groupby('AAABBBCCAAA'):
    print(key, list(group))
# A ['A', 'A', 'A']
# B ['B', 'B', 'B']
# C ['C', 'C']
# A ['A', 'A', 'A']     ← 注意：这是第二组 A，不是第一组的延续
```

**关键点一：`groupby` 只合并“相邻”的重复元素。** 上面字符串里 A 出现了两段，`groupby` 会给你**两个独立的 A 组**，它不会跨越中间的 B 和 C 去把 A 收拢到一起。如果你需要把全局相同的 key 归到一组，**必须先用 `sorted()` 按同一个 key 排序**，再 `groupby`。

**关键点二：`group` 是一个迭代器，必须“当场消费”。** 你在 `for` 循环里拿到 `group` 之后，**如果不在这一轮里把它转成 list 或遍历掉，下一轮循环开始时它就失效了**，因为 `groupby` 内部共享同一个底层迭代器。所以标准写法是 `list(group)` 当场转掉。

`keyfunc` 不传时，默认用元素本身作为 key。原文的忽略大小写例子：`lambda c: c.upper()`，让 `'A'` 和 `'a'` 返回同一个 key，就能归到一组。

---

## 组合迭代器（原文未提，拓展补充）

`itertools` 里还有一组专门做**排列组合**的函数，原文这页没有覆盖，但实际写算法题或生成测试用例时会用到，简单列一下：

- **`product(iterable, repeat=n)`**：笛卡尔积。`product('AB', repeat=2)` 得到 `AA, AB, BA, BB`。
- **`permutations(iterable, r)`**：排列，**顺序有关**。`permutations('ABC', 2)` 得到 `AB, AC, BA, BC, CA, CB`。
- **`combinations(iterable, r)`**：组合，**顺序无关**。`combinations('ABC', 2)` 得到 `AB, AC, BC`。

这三个函数**返回的也都是迭代器**，和前面所有函数一样，用时才生产。

---

## 实战：用 `count` + `takewhile` + `chain` 算 π

原文最后的练习是把前面几个函数串起来用，思路很清晰：

```python
import itertools

def pi(N):
    odds = itertools.count(1, 2)                          # 1, 3, 5, 7, ... 奇数序列
    ns = itertools.takewhile(lambda x: x <= 2*N - 1, odds) # 截取前 N 项
    result = 0
    for i, x in enumerate(ns):                            # 正负交替：4/1 - 4/3 + 4/5 ...
        result += 4 / x * ((-1) ** i)
    return result
```

这个例子的价值不在于 π 本身，而在于让你看到：**`count` 负责无限生产，`takewhile` 负责截断，for 循环负责消费**。三个环节各司其职，就是 itertools 的标准用法。

---

## 几个必须记住的坑

**1. 迭代器是“一次性”的。** 一个迭代器被消费完之后，**不能回头重新迭代**。你想用两次，就得重新创建一次 `itertools.count(...)`。

**2. `groupby` 不排序就是半成品。** 只合并相邻元素，要全局分组必须先排序。

**3. `group` 必须在当轮消费。** 不在 `for` 循环里当场 `list(group)`，下一轮它就空了。

**4. `repeat` 不传次数就是无限。** `repeat('A')` 会永远重复下去，如果不配 `takewhile` 或 `islice`，`list()` 会直接卡死。

---

## 自测

1. `itertools.count(1)` 创建的时候，内存里有没有 1、2、3、4……？为什么？
2. `takewhile` 和 `filter` 在处理无限序列时，根本区别是什么？
3. `groupby('AABBAA')` 会产出几组？为什么不是两组？
4. `groupby` 返回的 `group` 为什么必须当场消费？不消费会怎样？
5. `chain('ABC', 'XYZ')` 等价于什么样的嵌套循环？
6. 如果你要生成 `'ABCD'` 的所有两两组合（顺序无关），应该用 `itertools` 里的哪个函数？