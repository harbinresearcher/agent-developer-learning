> **重要程度：高**｜**手练：需要**
> 重点：只读属性、setter 校验和封装。

有没有既能检查参数，又可以用类似属性这样简单的方式来访问类的变量呢？对于追求完美的Python程序员来说，这是必须要做到的！

还记得装饰器（decorator）可以给函数动态加上功能吗？对于类的方法，装饰器一样起作用。Python内置的`@property`装饰器就是负责把一个方法变成属性调用的：
```python
class Student(object):
    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

要特别注意：属性的方法名不要和实例变量重名。例如，以下的代码是错误的：

```python
class Student(object):
    # 方法名称和实例变量均为birth:
    @property
    def birth(self):
        return self.birth
```

## 工作中注意

只定义 getter 时，属性默认不可赋值；需要写入校验时再定义 setter。属性内部一般使用 _score 等不同名称保存值，避免递归。属性访问应保持轻量、行为可预期；耗时 IO 或明显有副作用的操作更适合显式方法。

手练：创建带范围校验的属性，分别验证合法值、越界值和只读属性赋值。
