这节是数据库部分的**终极大招**。前面用 SQLite 和 MySQL 时，你都得手写 SQL 字符串；而 SQLAlchemy 要做的，就是让你**彻底摆脱手写 SQL，用纯 Python 对象的方式去操作数据库**。

---

# SQLAlchemy (ORM) · 笔记

## 先聊痛点：不用 ORM 时的“割裂感”

用原生 API 查数据库，`fetchall()` 返回的是元组列表，比如 `[('1', 'Michael'), ('2', 'Bob')]`。你拿到元组后，只能用 `row[0]`、`row[1]` 去猜哪个是 id、哪个是 name，代码很难读。并且，你一边写 Python，一边脑子里还要转 SQL，这两种思维的切换非常割裂。

SQLAlchemy 解决的正是这个问题，它的核心思想就是**把数据库的“表”映射成 Python 的“类”，把“行”映射成“对象”**。

## 定海神针：三步走骨架

无论用 SQLAlchemy 做什么，这三步地基必须打牢。

### 第一步：定义模型（Model），表就长在类里

你不需要再写 `CREATE TABLE`，只要写一个类继承 `Base`，用类属性描述字段：

```python
from sqlalchemy.orm import declarative_base
from sqlalchemy import Column, String, Integer

Base = declarative_base() # 所有模型的基类

class User(Base):
    __tablename__ = 'user' # 指定数据库中对应的表名

    id = Column(String(20), primary_key=True) # 定义字段 id，字符串，主键
    name = Column(String(20))                 # 定义字段 name，字符串
```
### 第二步：创建引擎（Engine），连上数据库

`create_engine` 负责处理底层连接。它的参数是一个**连接字符串**，格式固定为 `数据库类型+驱动://用户名:密码@地址:端口/库名`：

```python
from sqlalchemy import create_engine

# 注意：这里用的还是之前 MySQL 的连接方式
engine = create_engine('mysql+mysqlconnector://root:password@localhost:3306/test')
```

### 第三步：开会话（Session），拿上“暂存区”

`Session` 是所有数据库操作的入口。你通过它增删改查，它就像一个“暂存区”，你扔进去的对象，它最后帮你统一提交：

```python
from sqlalchemy.orm import sessionmaker

DBSession = sessionmaker(bind=engine) # 创建 Session 工厂
session = DBSession()                 # 获取一个具体的 Session 对象
```

## 核心操作：增、查、改、删

有了 Session，你操作的就是 Python 对象，不用再写 SQL 字符串。

### 增：造对象 -> add -> commit

```python
new_user = User(id='5', name='Bob') # 1. 造一个 User 对象
session.add(new_user)                # 2. 添加到 Session 暂存区
session.commit()                     # 3. 提交，真正写入数据库
session.close()                      # 4. 关闭 Session 释放资源
```
### 查：用 `select()` 代替 `query()`（**现代核心变化**）

这是你学这节最重要的知识点。廖雪峰教程里的 `session.query(User).filter(...)` 属于 **SQLAlchemy 1.x 老语法**。在 2.0 版本中，这套 API 已被官方标记为“遗留”，**新的标准写法是 `select()` 搭配 `session.execute()`**。

**1.x 老写法（廖雪峰教程）**：
```python
user = session.query(User).filter(User.id == '5').one()
```
**2.0 新写法（你必须掌握）**：
```python
from sqlalchemy import select

stmt = select(User).where(User.id == '5') # 1. 构造 select 语句
user = session.execute(stmt).scalar_one() # 2. 执行并提取单个对象
```

**为什么必须换**：2.0 的 `select()` 更贴近 SQL 的 `SELECT` 语义，并且统一了 Core 和 ORM 的查询方式，官方已明确未来的方向是 `select()`。你用 FastAPI 时，教程和社区代码也都会以这套新写法为准。

### 改：查出对象 -> 改属性 -> commit

```python
user = session.execute(select(User).where(User.id == '5')).scalar_one()
user.name = 'Alice' # 直接把对象的属性改掉
session.commit()     # commit 后自动将修改同步到数据库
```
### 删：查出对象 -> delete -> commit

```python
user = session.execute(select(User).where(User.id == '5')).scalar_one()
session.delete(user) # 标记删除
session.commit()     # 提交，真正删除
```

## 几个必须盯死的坑

**① `scalar_one()` 和 `scalars().all()` 的区别**
`scalar_one()` 要求**结果必须恰好有一个**，没有或超过一个都会报错；`scalars().all()` 返回所有对象的列表。如果你确定只有一个结果，用 `scalar_one()` 更安全。

**② 用完必须 `session.close()`**
Session 会占用数据库连接，用完不关会导致连接池被耗尽，程序卡死。推荐用 `with` 上下文管理器自动关闭：

```python
with DBSession() as session:
    user = session.execute(select(User).where(User.id == '5')).scalar_one()
    print(user.name)
# 出了 with 块，session 自动 close
```

**③ 一对多关系：用 `relationship` 定义，但不是数据库字段**

廖雪峰教程提到 `User` 拥有多个 `Book`，你可以在 `User` 类里加一行 `books = relationship("Book")`。**注意，这行不是数据库列，它只是 ORM 层面的便利属性**，让你能从 `user.books` 直接拿到关联对象列表，底层对应的还是外键 `user_id`。数据库里真正的关联字段，仍然要用 `ForeignKey` 在 `Book` 类里定义。

## 这页记住三件事

1. **ORM = 表映射成类、行映射成对象**，你不再写 SQL，而是操作 Python 对象。
2. **三步骨架：定义 Model（继承 Base）→ 创建 Engine（连接字符串）→ 开会话 Session**。
3. **查询用 `select()` + `session.execute()`（2.0 新写法）**，别再用 `session.query()` 老写法；增删改都是“操作对象 + `commit()`”。

---

## 自测

1. ORM 把数据库的“表”和“行”分别映射成了什么？解决了原生 DB-API 返回元组时什么痛点？
2. 定义模型时，`__tablename__` 和 `Column` 分别指定什么？`declarative_base()` 的作用是什么？
3. 为什么说 `Session` 是“暂存区”？`add` 之后不 `commit` 会怎样？
4. **2.0 版本中，`session.query(User)` 的替代写法是什么？** `scalar_one()` 和 `scalars().all()` 在使用场景上有什么区别？
5. `relationship` 定义的是数据库字段吗？真正的关联字段用什么来定义？