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

### 类作为类型[¶](https://fastapi.tiangolo.com/zh/python-types/#classes-as-types "Permanent link")

你也可以把类声明为变量的类型。

假设你有一个名为 `Person` 的类，带有 name：

`class Person:     def __init__(self, name: str):        self.name = name def get_person_name(one_person: Person)`



## Pydantic 模型[¶](https://fastapi.tiangolo.com/zh/python-types/#pydantic-models "Permanent link")

[Pydantic](https://pydantic.dev/docs/) 是一个用于执行数据校验的 Python 库。

**把"数据长什么样"用类声明出来，它帮你校验、帮你转换、不符合就直接报错**

**元数据就是"描述数据的数据"，是附加在数据身上的说明信息，它本身不是你要用的那个核心数据。**
放到 Python 类型提示里，`int` 是类型，是核心。但我想额外标注一句"这个整数必须大于 0"、"这个字符串最长 50 个字"、"这是查询参数不是请求体"——这些附加说明，就是给类型贴的**元数据**。


## Annotated
Python 原本的类型提示只能标类型，没法附带说明。`Annotated` 就是官方给的一个工具，让你**在类型旁边塞额外的元数据**。
![[{DF5AB755-2EFB-4054-A082-D1DB87B806B8}.png]]注意看，方括号里**第一个永远是真正的类型**，后面可以跟任意多个元数据
