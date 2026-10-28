这节原文的重点非常明确：前半截教你怎么把 MySQL 服务器跑起来，后半截教你用 Python 驱动去连它。由于你之前定过基调“MySQL 了解即可”，咱们把篇幅控住，只抓**必须知道的差异**和**实际会踩的坑**。

---

# 使用 MySQL · 笔记

## 先定性：它和 SQLite 分工不同

原文开篇说得很直接：**SQLite 是轻量级、可嵌入的，适合桌面和移动应用，但扛不住高并发；MySQL 是为服务器端设计的，能承受高并发，但内存占用远大于 SQLite**。

所以选型逻辑很简单：本地小工具、测试用例用 SQLite；线上 Web 服务用 MySQL。

---

## 第一步：把服务器跑起来（两种方式，选一种）

**直接安装**：官网下 Community Server 8.x，装的时候会提示设 root 密码。

**Docker 启动**（更省事，不用污染本机环境）：

```bash
docker run -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 --name mysql-8.4 -v ./mysql-data:/var/lib/mysql mysql:8.4 --mysql-native-password=ON --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

这段命令里你只需要盯住两个参数：**`-v` 把数据目录映射到本机**，避免容器一停数据全没；**`--character-set-server=utf8mb4`** 指定字符集，后面细说。看到输出最后一行 `ready for connections` 就表示启动成功。

---

## 核心坑一：编码必须用 `utf8mb4`，别用 `utf8`

这是原文反复强调的硬规矩。**MySQL 里的 `utf8` 是个历史坑，它最多只支持 3 个字节，存不了 emoji 和很多生僻字**；`utf8mb4` 才是真正完整的 UTF-8，支持最新的 Unicode 标准。

Windows 安装时直接选 UTF-8 编码；Mac/Linux 要改配置文件 `/etc/my.cnf`，把 `character-set-server` 设为 `utf8mb4`，然后重启 MySQL。

验证方法：进入 MySQL 命令行后执行 `show variables like '%char%';`，**看到 `character_set_client`、`character_set_database` 这些全是 `utf8mb4` 才算对**。有一个是 `utf8`，将来就可能出现乱码或插入失败。

**关键认知**：如果你的 MySQL 版本低于 5.5.3，那就只能用 `utf8`，`utf8mb4` 是 5.5.3 之后才支持的。现在新装基本都是 8.x，无脑选 `utf8mb4` 就行。

---

## 核心坑二：Python 的占位符是 `%s`，不是 `?`

这是从 SQLite 切换过来最容易写错的地方。

**SQLite** 的占位符是 `?`，**MySQL 的占位符是 `%s`**。把 SQLite 的代码直接搬过来，`execute` 会直接报错。

```python
# 插入
cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'Michael'])

# 查询
cursor.execute('select * from user where id = %s', ('1',))
```

**传参用列表或元组都行**，但占位符必须写成 `%s`。这里的 `%s` 不是 Python 字符串格式化的那个 `%s`，而是 MySQL 驱动识别的参数标记，不要混淆。

---

## Python 怎么连：驱动和通用 API

Python 官方提供的驱动是 `mysql-connector-python`，直接 `pip install mysql-connector-python`。

连接代码和 SQLite 几乎一模一样，只是换了个模块名：

```python
import mysql.connector

conn = mysql.connector.connect(
    user='root',
    password='password',   # 填你安装时设的 root 密码
    database='test'        # 连到 test 这个数据库，要先自己建好
)
cursor = conn.cursor()
```

**为什么代码能这么像**：Python 有一套 **DB-API 标准**，规定了 `connect()`、`cursor()`、`execute()`、`commit()`、`fetchall()` 这些方法的名字。SQLite、MySQL、PostgreSQL 的驱动都遵循这套标准，所以你换数据库时，**上层业务代码几乎不用改，只换驱动和占位符**就行。

---

## 和 SQLite 完全一致的部分：别重复踩

- **写完必须 `conn.commit()`**：`execute` 只把数据放在内存里，不 commit 就关连接，数据直接丢。
- **查完用 `fetchall()`**：返回元组列表，一行一个元组。
- **资源必须关**：`cursor.close()` 和 `conn.close()` 在 `finally` 里兜底，别裸奔。

---

## 一句话收束

**MySQL 是服务器端的数据库，装好之后编码统一设成 `utf8mb4`；Python 驱动用 `mysql-connector-python`，连接和操作代码跟 SQLite 一样，唯独占位符从 `?` 换成 `%s`。** 安装配置的细节用到时再翻，代码层面的差异就这一条。