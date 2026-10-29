> **重要程度：中**｜**手练：建议**
> 重点：线程本地变量、生命周期及 contextvars 区别。


## 先看它解决什么痛点：每个函数都要传参

假设你有一个处理学生数据的任务，每个线程处理一个学生。最朴素的做法是把 `student` 对象一层层往下传：

```python
def process_student(name):
    std = Student(name)      # 创建 student 对象
    do_task_1(std)           # 传给第一层任务
    do_task_2(std)           # 传给第二层任务

def do_task_1(std):
    do_subtask_1(std)        # 还得继续往下传
    do_subtask_2(std)

def do_task_2(std):
    do_subtask_2(std)        # 每一层都得接住再传下去
```

**问题**：`std` 从 `process_student` 一路传到最底层的子任务，中间每一层函数都被迫加一个参数，哪怕它自己根本不用 `std`。函数调用链越深，参数传递越像传接力棒，代码全是噪音。

你可能会想：**把 `std` 设成全局变量不就行了？** 不行。因为**每个线程处理的是不同的学生对象**，线程 A 的 `std` 和线程 B 的 `std` 必须是两个不同的东西，共享一个全局变量就串了。

一个“看起来能用”但很丑的办法是：**用一个全局 dict，以线程对象为 key**：

```python
global_dict = {}                                   # 全局字典，所有线程共用

def std_thread(name):
    std = Student(name)                            # 创建本线程的 student
    global_dict[threading.current_thread()] = std  # 以当前线程为 key 存进去
    do_task_1()                                    # 下面调用不用再传 std 了

def do_task_1():
    std = global_dict[threading.current_thread()]  # 每个函数自己去 dict 里捞
    ...
```

这个方案**逻辑上行得通**，确实消灭了参数传递。但每个函数里都要写一句 `global_dict[threading.current_thread()]`，又丑又重复。ThreadLocal 就是把这句话**封装起来**，让你用起来像在写一个普通的全局变量。

---

## ThreadLocal 的直觉：一个“会认人”的全局变量

```python
import threading

local_school = threading.local()                   # 创建一个全局的 ThreadLocal 对象

def process_student():
    std = local_school.student                     # 读当前线程自己的 student
    print(f'Hello, {std} (in {threading.current_thread().name})')

def process_thread(name):
    local_school.student = name                    # 写当前线程自己的 student
    process_student()

t1 = threading.Thread(target=process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target=process_thread, args=('Bob',), name='Thread-B')
t1.start(); t2.start(); t1.join(); t2.join()
# 输出：Hello, Alice (in Thread-A)  /  Hello, Bob (in Thread-B)
```

`local_school` 是一个**全局对象**，所有线程都能访问它。但神奇的地方在于：**你在 Thread-A 里写 `local_school.student = 'Alice'`，在 Thread-B 里写 `local_school.student = 'Bob'`，两者完全互不干扰**，每个线程读到的都是自己写进去的那个值。

你可以把 `local_school` 想象成一个**会根据当前线程自动切换内容的全局变量**：同一个名字，线程 A 看到的是 A 的副本，线程 B 看到的是 B 的副本。线程本地属性分别保存，但若不同线程把同一个可变对象存进去，对象本身仍然共享，需要适当同步。

---

## 它背后的原理：还是那个“以线程为 key 的 dict”

ThreadLocal 内部干的事，本质上就是前面那个“丑办法”的自动化版本：**它自己维护了一个以线程标识为 key 的字典**。你访问 `local_school.student` 时，它自动根据当前线程找到对应的存储槽，把值塞进去或读出来。你不需要自己拿 `threading.current_thread()` 去查，ThreadLocal 替你做了。

所以记住一句话：ThreadLocal 提供按线程隔离的属性存储，不会自动让存入的共享对象变得线程安全。你声明一个全局对象，但每个线程对它属性的读写都是独立的。

---

## 常见坑：一个 ThreadLocal 对象，只创建一次

这是最容易出问题的地方。需要在多个函数之间共享同一个线程本地容器时，应让它们访问同一个 `threading.local()` 实例，常见做法是在模块级创建：

```python
# 正确：模块级别创建一次
local_data = threading.local()

def worker():
    local_data.x = 1      # 用的是同一个 ThreadLocal 对象

# 错误：每次调用都新建一个 ThreadLocal
def worker_bad():
    local = threading.local()   # 每次进来都新建，失去了“全局可访问”的意义
    local.x = 1
```

如果你在每个函数里都 `threading.local()`，每个线程拿到的都是一个全新的对象，**根本起不到“线程间隔离、但同一线程内共享”的效果**，还不如直接用普通局部变量。

---

## 另一个限制：它只管线程，不管协程

ThreadLocal 的隔离单位是**操作系统线程**。但在现代 Python 里，尤其是 `asyncio` 的异步代码中，**多个协程可能跑在同一个线程上**。这时 ThreadLocal 会把它们当成同一个“线程”，导致一个协程写的数据，被另一个协程读到，隔离失效。

Python 3.7 引入了 `contextvars` 来解决这个问题。它的隔离单位是**上下文**（协程 / 异步任务），而不是线程。在异步代码里，应该用 `contextvars.ContextVar` 代替 `threading.local()`。

**怎么选**：

- 纯多线程代码（`threading`）→ `threading.local()` 够用，API 简单；
- 异步代码（`asyncio`）或线程池 + 协程混合 → 用 `contextvars`，否则数据会串。

---

## 这页记住三件事

1. **ThreadLocal 是一个全局对象，但每个线程对它的属性读写是独立的副本**，不用传参、不用加锁。
2. **它内部就是一个以线程为 key 的 dict**，只是把“自己去查”封装成了“像用普通属性一样用”。
3. **`threading.local()` 在模块级别创建一次**，反复创建等于白搭；**异步代码里它不够用**，换 `contextvars`。

---

## 自测

1. 不用 ThreadLocal 时，为什么“全局 dict + 线程作 key”能解决问题？它丑在哪？
2. `local_school.student` 在线程 A 和线程 B 里互不干扰，ThreadLocal 内部大致是怎么做到的？
3. 为什么 `threading.local()` 不能在函数里反复调用？反复调用会失去什么？
4. ThreadLocal 在 `asyncio` 代码里为什么可能失效？应该用什么替代？
线程池会复用线程，线程本地值可能留到后续任务。每个任务应明确初始化和清理；异步请求上下文优先考虑 `contextvars`，并在任务结束时恢复上下文。