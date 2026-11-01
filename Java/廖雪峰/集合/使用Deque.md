# 使用Deque

> **重要程度：高**｜**手练：需要**
> **重点**：双端操作、队列和栈、空值。

## 解决什么问题

`Deque` 是双端队列，可以在首尾两端添加和取出。按一端进另一端出可当队列；按同一端进出可当栈。`ArrayDeque` 是常见实现，适合单线程或受外部同步保护的场景。

## 完整例子

保存为 `DequeDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;

public class DequeDemo {
    private static final Logger logger = Logger.getLogger(DequeDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：从两端操作队列
         * ========================================================================
         * 数据源：程序中准备的示例数据
         * 操作：
         * 1) 创建集合并填入元素
         * 2) 按规则读取元素并输出
         */
        logger.info("开始处理集合");
        // 1.1 在两端添加元素
        Deque<String> deque = new ArrayDeque<>();
        deque.offerLast("B");
        deque.offerFirst("A");
        deque.offerLast("C");
        // 1.2 从首尾取出，并观察剩余队首
        System.out.println("first=" + deque.pollFirst());
        System.out.println("last=" + deque.pollLast());
        System.out.println("left=" + deque.peekFirst());
        logger.info("集合处理完成");
    }
}
```

标准输出：

```text
first=A
last=C
left=B
```

## 执行过程

加入 B 后，队列为 `[B]`；在首部加入 A 得到 `[A, B]`；在尾部加入 C 得到 `[A, B, C]`。`pollFirst` 取出 A，`pollLast` 取出 C，`peekFirst` 只查看剩下的 B。

## 方法和边界

`offerFirst/offerLast` 从指定端添加；`pollFirst/pollLast` 取出并删除，空时返回 `null`；`peekFirst/peekLast` 查看，空时返回 `null`。对应的 `removeFirst/removeLast` 在空时抛异常。

栈操作 `push` 等价于添加到首部，`pop` 从首部移除且空时抛异常。需要空时返回 `null`，可用 `pollFirst`。`ArrayDeque` 禁止 null，也不保证并发访问安全；限容量队列、阻塞和多线程语义需选择其他实现，不能仅凭接口名假设。

## 手练与答案

对空 deque 执行 `push("A")`、`push("B")`、`pop()`：返回 B，体现后进先出。接着 `pollLast()` 返回 A；再 `pollFirst()` 返回 null。

参考：[教程：使用Deque](https://liaoxuefeng.com/books/java/collection/deque/index.html)；[Oracle 文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Deque.html)。
