> **重要程度：高**｜**手练：需要**
> 重点：时区感知时间、UTC、格式转换、时间差。

# datetime · 笔记

## 先聊一个最基础的困惑：模块和类同名

你 `from datetime import datetime`，导进来的其实是 **`datetime` 模块里的 `datetime` 类**，不是模块本身。`datetime` 是模块名，里面还装着 `timedelta`、`timezone` 这些东西；而 `datetime` 类才是你真正用来“表示一个时刻”的那个东西。如果你只写 `import datetime`，那用的时候必须写全名 `datetime.datetime.now()`，多一层。日常**推荐 `from datetime import datetime`**，少打一截。

```python
from datetime import datetime          # 导入 datetime 类，不是模块
now = datetime.now()                   # 获取当前日期和时间
print(now)                             # 2015-05-18 16:28:07.198690
print(type(now))                       # <class 'datetime.datetime'>，类型就是 datetime
```

要指定某个时刻，直接构造：`datetime(2015, 4, 19, 12, 20)` → `2015-04-19 12:20:00`。很简单，不展开。

---

## 重头戏：timestamp，计算机眼里只有“一个数”

Unix 时间戳通常表示从 1970 年 1 月 1 日 00:00:00 UTC 起经过的秒数；1970 年以前通常是负数。时间还有其他存储格式，不能笼统说计算机只存时间戳。

关键认知在这里：**timestamp 和时区完全无关**。`timestamp = 0` 对全世界所有计算机都是同一个时刻，只不过北京人看到的是 `1970-1-1 08:00:00`（UTC+8），伦敦人看到的是 `1970-1-1 00:00:00`（UTC+0）。**同一个 timestamp，换算成不同时区的“墙上时间”不同，但它本身是唯一的。** 这就是为什么全球计算机对表都用 timestamp。

### datetime → timestamp

```python
dt = datetime(2015, 4, 19, 12, 20)     # 创建一个“本地时间”的 datetime
dt.timestamp()                          # 1429417200.0，转成 timestamp
```

**注意**：这里的 `dt` 是**本地时间**，`timestamp()` 会按你操作系统的时区去算。如果你在北京，`12:20` 是 UTC+8 的 12:20，算出来的 timestamp 和 UTC+0 的 12:20 不一样。**同一个 timestamp 全球唯一，但同一个“墙上时间”在不同时区对应的 timestamp 不同。**

Python 的 timestamp 是**浮点数**，整数位是秒。如果你从 JavaScript 拿到的是毫秒数，**除以 1000** 就是 Python 的表示。

### timestamp → datetime

本地时间可用 `datetime.fromtimestamp(t)`；需要带 UTC 时区的结果时，用 `datetime.fromtimestamp(t, timezone.utc)`。旧接口 `utcfromtimestamp()` 自 Python 3.12 起已弃用。

不传时区参数时，`fromtimestamp()` 返回的 datetime **没有时区信息**，它只是按本地时区给你一个“墙上时间”。

---

## str 和 datetime 互转：两个函数，方向别搞反

**str → datetime 用 `strptime`**，需要一个格式字符串告诉它“这个字符串的哪部分是年、哪部分是月”：

```python
cday = datetime.strptime('2015-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')
print(cday)                             # 2015-06-01 18:19:59，注意月份自动补零了
```

`%Y` 是四位年份，`%m` 是两位月份，`%d` 是两位日期，`%H` 是 24 小时制的小时，`%M` 是分钟，`%S` 是秒。**转换出来的 datetime 没有时区信息**，别指望它知道这是哪个时区的时间。

**datetime → str 用 `strftime`**，方向反过来，把 datetime 按你给的格式“印”成字符串：

```python
now.strftime('%a, %b %d %H:%M')        # 'Mon, May 05 16:28'
```

`%a` 是星期缩写，`%b` 是月份缩写。**`strptime` 的 p 代表 parse（解析字符串），`strftime` 的 f 代表 format（格式化输出）**，记住方向，别写反。

---

## datetime 加减：用 timedelta，别手动算

```python
from datetime import datetime, timedelta

now + timedelta(hours=10)              # 10 小时后
now - timedelta(days=1)                # 1 天前
now + timedelta(days=2, hours=12)      # 2 天 12 小时后
```

`timedelta` 可以接受 `days`、`hours`、`minutes`、`seconds` 等参数，**直接和 datetime 做 `+` / `-`**，得到一个新的 datetime。

**但有一个硬限制**：`timedelta` 里的 `days` 是**固定的 24 小时**，它不懂“一个月有 28/29/30/31 天”。你要算“一个月后”，不能用 `timedelta(days=30)`，必须用 `dateutil.relativedelta` 或者自己处理月份进位。**`timedelta` 只适合“固定长度的时间段”**，涉及月份、年份的加减它管不了。

---

## 时区：默认没有，必须显式设

`datetime` 对象有一个 `tzinfo` 属性，**默认是 `None`**。这意味着 `datetime.now()` 给你的是一个**“裸”的本地时间**，它自己不知道是哪个时区，你也没法拿它做跨时区比较。

要给 datetime 加上时区，用 `timezone` + `replace`：

```python
from datetime import datetime, timedelta, timezone

tz_utc_8 = timezone(timedelta(hours=8))    # 创建一个 UTC+8 的时区对象
dt = datetime.now().replace(tzinfo=tz_utc_8)   # 强制把这个时间标记为 UTC+8
```

**但 `replace(tzinfo=...)` 不会帮你换算时间**，它只是给现有的“墙上时间”贴一个时区标签。**如果系统时区不是 UTC+8，你硬贴 UTC+8 标签，结果是错的**——时间没变，但含义变了。

### 正确的时区转换方式：`astimezone`

要真正做时区转换，流程是：**先拿到一个带时区的 datetime 作为基准，再用 `astimezone()` 转到目标时区**。

先用 `datetime.now(timezone.utc)` 得到带 UTC 时区的当前时间，再调用 `astimezone(目标时区)`。

**`astimezone()` 会真正换算时间**，不是贴标签。而且**不要求从 UTC 出发**，任何带时区的 datetime 都能直接转到另一个时区，例如可以先转到北京时间，再转到东京时间。

**时区转换的唯一关键**：拿到一个 datetime 时，**你必须知道它原本是哪个时区**。不知道原始时区，转换就是猜。所以要么从 timestamp 转（timestamp 表示时间点，本身不携带时区名称），要么用 `datetime.now(timezone.utc)` 从 UTC 出发。

---

## 原文小结里最值钱的两句话

第一句：**“datetime 表示的时间需要时区信息才能确定一个特定的时间，否则只能视为本地时间。”** 一个没有 `tzinfo` 的 datetime，只是一个“墙上时间的快照”，它不指向全球唯一的时刻。

第二句：存储和传输时间点时，可使用 UTC 感知 datetime 或明确单位的 Unix 时间戳，并约定格式和精度。需要保留原始地区或未来日程时，还应保存 IANA 时区，例如 `Asia/Shanghai`；固定 UTC 偏移不能表达夏令时规则。Python 3.12 起 `utcnow()` 已弃用，改用 `datetime.now(timezone.utc)`。参考：[datetime](https://docs.python.org/3/library/datetime.html)。

---

## 自测

1. `from datetime import datetime` 和 `import datetime` 之后调用方式有什么不同？为什么推荐前者？
2. 同一个“墙上时间” `2024-01-01 12:00:00`，在北京和伦敦对应的 timestamp 相同吗？为什么？
3. `datetime.fromtimestamp(t)` 和 `datetime.fromtimestamp(t, timezone.utc)` 在时区信息上有什么区别？
4. `strptime` 和 `strftime` 哪个是“字符串转 datetime”，哪个是反方向？`%Y` 和 `%m` 分别代表什么？
5. 为什么 `timedelta(days=30)` 不能可靠地表示“一个月后”？
6. `replace(tzinfo=...)` 和 `astimezone(...)` 的本质区别是什么？做时区转换时，为什么必须先知道原始时区？
7. 原文建议存 datetime 时转成 timestamp，理由是什么？