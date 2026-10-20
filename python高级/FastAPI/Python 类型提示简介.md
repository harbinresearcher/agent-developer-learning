Python 支持可选的“类型提示”（也叫“类型注解”）。

这些 **“类型提示”** 或注解是一种特殊语法，用来声明变量的类型。通过为变量声明类型，编辑器和工具可以为你提供更好的支持。

简单说 类型注解有利于帮助我们正确的补全

这和声明默认值不同，比如：

    `first_name="john", last_name="doe"`

这是两码事。

我们用的是冒号（`:`），不是等号（`=`）。

而且添加类型提示通常不会改变代码本来的行为。


### `typing` 模块

在一些额外的用例中，你可能需要从标准库的 `typing` 模块导入内容。比如当你想声明“任意类型”时，可以使用 `typing` 中的 `Any`：
![[{B41224D0-F004-4693-B032-5BF32E0E49C4}.png]]


### 泛型类型[¶](https://fastapi.tiangolo.com/zh/python-types/#generic-types "Permanent link")

有些类型可以在方括号中接收“类型参数”（type parameters），用于声明其内部值的类型。比如“字符串列表”可以写为 `list[str]`。

这些能接收类型参数的类型称为**泛型类型**（Generic types）或**泛型**（Generics）。

![[{D6B8857A-2561-418D-98B5-A1E7B681DB3B}.png]]
方括号中的这些内部类型称为“类型参数”（type parameters）。

在这个例子中，`str` 是传给 `list` 的类型参数。

#### 元组和集合[¶](https://fastapi.tiangolo.com/zh/python-types/#tuple-and-set "Permanent link")

声明 `tuple` 和 `set` 的方式类似：

`def process_items(items_t: tuple[int, int, str], items_s: set[bytes]):     return items_t, items_s`

这表示：

- 变量 `items_t` 是一个含有 3 个元素的 `tuple`，分别是一个 `int`、另一个 `int`，以及一个 `str`。
- 变量 `items_s` 是一个 `set`，其中每个元素的类型是 `bytes`。

#### 字典[¶](https://fastapi.tiangolo.com/zh/python-types/#dict "Permanent link")

定义 `dict` 时，需要传入 2 个类型参数，用逗号分隔。

第一个类型参数用于字典的键。

第二个类型参数用于字典的值：

def process_items(prices: dict[str, float]):



#### Union[¶](https://fastapi.tiangolo.com/zh/python-types/#union "Permanent link")

你可以声明一个变量可以是**若干种类型**中的任意一种，比如既可以是 `int` 也可以是 `str`。

定义时使用竖线（`|`）把两种类型分开。

这称为“联合类型”（union），因为变量可以是这两类类型集合的并集中的任意一个。

`def process_item(item: int | str):


#### 可能为 `None`[¶](https://fastapi.tiangolo.com/zh/python-types/#possibly-none "Permanent link")

你可以声明一个值的类型是某种类型（比如 `str`），但它也可能是 `None`。

`def say_hi(name: str | None = None):
