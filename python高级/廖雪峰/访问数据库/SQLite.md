这节原文虽然只有几十行代码，但它把你后面用 SQLAlchemy 甚至 FastAPI 操作数据库时**最底层的机制**全给亮了底。咱们顺着“它是个啥 → 怎么连 → 怎么建表/写 → 怎么查 → 怎么防坑”这条线走，把底层逻辑摸透。

---

# 使用 SQLite · 笔记

## 先定性：它不是一个“服务器”，就是一个文件

SQLite 是一种**嵌入式数据库**，它的数据库**本身就是一个文件**（比如 `test.db`）。你不需要像 MySQL 那样去启动一个后台服务，Python 直接就能读写这个文件。而且 Python 内置了 `sqlite3` 驱动，**不需要安装任何东西**，拿来就能用。

## 铁三角：Connection、Cursor 和驱动

这是所有 Python 数据库操作的地基。你要用 Python 操作任何数据库，都要遵循这套标准 API。

- **Connection（连接）**：代表和数据库的物理连接，比如 `conn = sqlite3.connect('test.db')`。如果文件不存在，它会自动创建一个。
- **Cursor（游标）**：连接建立了只是通了路，你要真正执行 SQL，必须拿到游标。`cursor = conn.cursor()`。**所有的 SQL 语句都是通过游标的 `execute()` 方法发给数据库的**。

## 写数据：执行完必须 Commit

建表和插入数据的代码是这样的：

```python
import sqlite3                              # 导入内置驱动
conn = sqlite3.connect('test.db')           # 连接数据库，文件不存在会自动建
cursor = conn.cursor()                      # 打开游标，准备执行SQL

# 建一张 user 表
cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')  

# 插入一条记录
cursor.execute('insert into user (id, name) values (\'1\', \'Michael\')')  

print(cursor.rowcount)                      # 1，rowcount 告诉你插入了几行
conn.commit()                               # 提交事务，数据才真正写入文件！
```

**核心干货**：`execute` 执行完之后，数据只是**暂存**在内存里。**必须调 `conn.commit()`**，数据才会写进硬盘文件。不 commit 就关连接，数据直接丢，这是新手最容易踩的坑。

## 查数据：fetchall 返回的是元组列表

查询代码稍微不同，`execute` 之后需要调 `fetchall()` 来取结果：

```python
conn = sqlite3.connect('test.db')           # 重新连
cursor = conn.cursor()                      # 开游标
cursor.execute('select * from user where id=?', ('1',))   # 执行查询，带参数
values = cursor.fetchall()                  # 获取结果集
print(values)                               # [('1', 'Michael')]
```

**核心干货**：`fetchall()` 返回的是一个 **list**，里面每一个元素都是一个 **tuple**，对应数据库里的一行记录。

## 防注入：为什么 SQL 里要写 `?`

原文专门强调了 SQL 带参数时的写法：**千万不要用字符串拼接**，要用 `?` 占位符，然后把参数作为元组传给 `execute`。

```python
# 正确写法：用 ? 占位
cursor.execute('select * from user where id=?', ('1',))  

# 错误写法：千万别这么干！
cursor.execute(f'select * from user where id={id}')       # 会被 SQL 注入攻击
```

**为什么？** 如果你用字符串拼接，用户输入恶意内容（比如 `'1' OR '1'='1'`），就能把整个表的数据全查出来。**`?` 占位符会帮你做转义**，这是最基础的防御手段。

## 资源释放：用 try...finally 或 with 语句

原文明确提醒：**打开的 Connection 和 Cursor 必须关闭，否则资源会泄露**。但直接写 `close()` 遇到异常就可能执行不到，所以要用 `try...finally` 兜底：

```python
conn = sqlite3.connect('test.db')
try:
    cursor = conn.cursor()
    cursor.execute('select * from user')
    values = cursor.fetchall()
finally:
    cursor.close()          # 不管有没有报错，一定关游标
    conn.close()            # 一定关连接
```

## 这页记住三件事

1. **SQLite 是一个文件**，Python 内置支持，不用装服务。
2. **Connection 负责连，Cursor 负责执行**；写完数据**必须 `commit()`**，查数据用 **`fetchall()`** 拿元组列表。
3. **SQL 参数用 `?` 占位**，别拼字符串；**打开的资源必须关**，用 `try...finally` 保证。

---

## 自测

1. SQLite 的数据库以什么形式存在？和 MySQL 有什么根本区别？
2. `conn.commit()` 不调会怎样？数据去哪了？
3. `fetchall()` 返回的是什么数据结构？取一行数据用什么方法？
4. SQL 语句里的 `?` 是干嘛的？不用它会有什么安全隐患？
5. 为什么 `cursor.close()` 和 `conn.close()` 要放在 `finally` 里？