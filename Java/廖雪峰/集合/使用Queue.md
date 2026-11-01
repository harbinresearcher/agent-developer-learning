# 使用Queue

> **重要程度：高**｜**手练：需要**
> **重点**：先进先出、offer/poll/peek 与空队列。

## 解决什么问题

队列保存等待处理的数据。先入先出（FIFO）队列会先处理先加入的元素，适合排队任务。`Queue<E>` 是接口，具体顺序由实现决定；并非所有 Queue 都是 FIFO，[[使用PriorityQueue]] 按优先级取出。

## 完整例子

保存为 `QueueDemo.java`，用 `javac -encoding UTF-8 QueueDemo.java` 编译，再运行 `java QueueDemo`。日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;

public class QueueDemo {
    private static final Logger logger = Logger.getLogger(QueueDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：队列入队与出队
         * ========================================================================
         * 数据源：程序中准备的示例数据
         * 操作：
         * 1) 创建集合并填入元素
         * 2) 按规则读取元素并输出
         */
        logger.info("开始处理集合");
        // 1.1 准备先进先出的队列
        Queue<String> queue = new ArrayDeque<>();
        queue.offer("A");
        queue.offer("B");
        // 1.2 查看、移除队首并检查空队列
        System.out.println("peek=" + queue.peek());
        System.out.println("first=" + queue.poll());
        System.out.println("second=" + queue.poll());
        System.out.println("empty=" + queue.poll());
        logger.info("集合处理完成");
    }
}
```

标准输出：

```text
peek=A
first=A
second=B
empty=null
```

## 执行过程

`offer("A")` 与 `offer("B")` 依次入队；`peek()` 只看队首 A，不删除。两次 `poll()` 依次删除并返回 A、B，第三次队列为空，返回 `null`。

## 常用操作和边界

- `offer` 添加；有容量限制且无法添加时通常返回 `false`。`add` 在容量不足时抛 `IllegalStateException`。
- `poll` 取出并删除队首，空时返回 `null`；`remove` 空时抛 `NoSuchElementException`。
- `peek` 查看但不删除，空时返回 `null`；`element` 空时抛 `NoSuchElementException`。

本例的 `ArrayDeque` 不允许 `null` 元素，所以 `poll` 返回 `null` 可明确表示空队列。具体实现的容量、顺序和空值策略要看 API。普通 `ArrayDeque` 非线程安全；生产者消费者跨线程协作可以研究 `BlockingQueue`，它提供等待和限时操作。

## 手练与答案

先 `offer("C")` 再 `peek()`：看到 A，C 留在队尾。清空后调用 `remove()`：抛 `NoSuchElementException`，与返回 null 的 `poll()` 不同。

参考：[教程：使用Queue](https://liaoxuefeng.com/books/java/collection/queue/index.html)；[Oracle 文档](https://docs.oracle.com/javase/tutorial/collections/interfaces/queue.html)。
