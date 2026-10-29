> **重要程度：高**｜**手练：需要**
> 重点：参数化查询、事务、索引和连接管理。

# 使用 MySQL · 笔记

## 先定性：它和 SQLite 分工不同

SQLite 是嵌入式数据库，适合本地工具、测试及部分服务器应用；MySQL 是独立的数据库服务，适合需要集中管理和多客户端访问的场景。

选择要看并发写入、部署方式、数据量、备份和运维要求，不能只按是否上线判断。

---

## 第一步：把服务器跑起来（两种方式，选一种）

**直接安装**：官网下 Community Server 8.x，装的时候会提示设 root 密码。

**Docker 启动**（更省事，不用污染本机环境）：

本地容器应固定项目认可的 MySQL 版本，配置持久化卷和仅供本机使用的连接凭据。具体启动参数以镜像文档和项目配置为准。

数据卷用于持久化数据库文件；容器运行与数据库已就绪是两件事，应检查健康状态和连接。不要把生产密码写进命令历史或源码。

---

## 核心坑一：编码必须用 `utf8mb4`，别用 `utf8`

这是原文反复强调的硬规矩。**MySQL 里的 `utf8` 是个历史坑，它最多只支持 3 个字节，存不了 emoji 和很多生僻字**；`utf8mb4` 才是真正完整的 UTF-8，支持最新的 Unicode 标准。

Windows 安装时直接选 UTF-8 编码；Mac/Linux 要改配置文件 `/etc/my.cnf`，把 `character-set-server` 设为 `utf8mb4`，然后重启 MySQL。

验证方法：进入 MySQL 命令行后执行 `show variables like '%char%';`，**看到 `character_set_client`、`character_set_database` 这些全是 `utf8mb4` 才算对**。有一个是 `utf8`，将来就可能出现乱码或插入失败。

新项目通常选 `utf8mb4`，并核对数据库、表、列及连接设置。`mysql_native_password` 在 MySQL 8.0.34 起弃用、8.4 默认禁用、9.0 移除；优先使用受支持的认证方式，不要为新项目启用旧插件。参考：[MySQL 认证文档](https://dev.mysql.com/doc/refman/8.4/en/native-pluggable-authentication.html)。

---

## 核心坑二：Python 的占位符是 `%s`，不是 `?`

这是从 SQLite 切换过来最容易写错的地方。

Python 内置 `sqlite3` 常用 `?` 占位；本页所用 MySQL 驱动常用 `%s`。占位符语法由具体驱动决定，不要自行拼接用户输入。

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

- **写完必须 `conn.commit()`**：事务未自动提交时，`execute` 已执行 SQL，但修改还需提交才能作为已提交结果保留；异常时回滚。自动提交行为取决于驱动和配置。
- **查完用 `fetchall()`**：返回元组列表，一行一个元组。
- **资源必须关**：`cursor.close()` 和 `conn.close()` 在 `finally` 里兜底，别裸奔。

---

## 一句话收束

**MySQL 是服务器端的数据库，装好之后编码统一设成 `utf8mb4`；Python 驱动用 `mysql-connector-python`，连接和操作代码跟 SQLite 一样，唯独占位符从 `?` 换成 `%s`。** 安装配置的细节用到时再翻，代码层面的差异就这一条。