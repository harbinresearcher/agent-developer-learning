# 使用Stack

> **重要程度：高**｜**手练：需要**
> **重点**：后进先出、Deque 替代旧 Stack。

## 解决什么问题

栈是后进先出（LIFO）：最后放入的元素先取出。方法调用、撤销记录、括号匹配等都可能用到栈思想。Java 保留旧的 `Stack` 类；新代码一般可以用 `Deque` 表达栈，常用实现为 `ArrayDeque`。

## 完整例子

保存为 `StackDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;

public class StackDemo {
    private static final Logger logger = Logger.getLogger(StackDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：用双端队列模拟栈
         * ========================================================================
         * 数据源：程序中准备的示例数据
         * 操作：
         * 1) 创建集合并填入数据
         * 2) 查询并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 依次入栈
        Deque<String> stack = new ArrayDeque<>();
        stack.push("A");
        stack.push("B");
        // 1.2 查看栈顶并依次出栈
        System.out.println("top=" + stack.peek());
        System.out.println("first=" + stack.pop());
        System.out.println("second=" + stack.pop());
        System.out.println("empty=" + stack.pollFirst());
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
top=B
first=B
second=A
empty=null
```

## 执行过程

先 `push("A")`，再 `push("B")`，B 处于栈顶。`peek()` 只查看 B；`pop()` 删除并返回 B，第二次 `pop()` 返回 A。栈空后，`pollFirst()` 返回 null；若此时再调用 `pop()`，会抛 `NoSuchElementException`。

## 工作中怎么用

`push`、`pop` 对应 deque 的首端操作，保持在同一端就有 LIFO 语义。`ArrayDeque` 禁止 null，且不是并发安全容器；跨线程共享时要选合适的同步方案。替换已有 `Stack` 时，检查旧代码是否依赖 `Vector` 提供的随机访问或同步方法，不能只换类型名就假定行为完全相同。

## 手练与答案

入栈 1、2、3 后依次弹出得到 3、2、1。先 `peek()` 不会减少栈大小；使用 `pop()` 才会移除栈顶。

参考：[教程：使用Stack](https://liaoxuefeng.com/books/java/collection/stack/index.html)；[Oracle 文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Deque.html)。
