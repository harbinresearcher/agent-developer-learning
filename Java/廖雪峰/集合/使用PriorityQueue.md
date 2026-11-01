# 使用PriorityQueue

> **重要程度：高**｜**手练：需要**
> **重点**：优先级、比较规则与非排序遍历。

## 解决什么问题

有些任务不能仅按到达时间处理，而要优先处理更紧急的任务。`PriorityQueue` 按元素的自然顺序或传入的比较器选择队首；默认数字顺序中，较小的数字先出队。

## 完整例子

保存为 `PriorityDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;

public class PriorityDemo {
    private static final Logger logger = Logger.getLogger(PriorityDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：按优先级处理任务
         * ========================================================================
         * 数据源：程序中准备的示例数据
         * 操作：
         * 1) 创建集合并填入元素
         * 2) 按规则读取元素并输出
         */
        logger.info("开始处理集合");
        // 1.1 放入不同优先级的数字
        Queue<Integer> tasks = new PriorityQueue<>();
        tasks.offer(5);
        tasks.offer(1);
        tasks.offer(3);
        // 1.2 逐个取出队首
        System.out.println(tasks.poll());
        System.out.println(tasks.poll());
        System.out.println(tasks.poll());
        logger.info("集合处理完成");
    }
}
```

标准输出：

```text
1
3
5
```

## 执行过程

先后加入 5、1、3。队列按比较规则选择最小元素，三次 `poll()` 分别取出 1、3、5。出队顺序与插入顺序不同。队列内部并不保证以完整排序形式保存或遍历；打印整个队列或用迭代器遍历不能用来验证有序出队。

## 选择比较规则

元素需能按自然顺序比较，或在创建队列时传入 `Comparator`。若多个元素比较结果相等，先取哪一个没有稳定保证；业务需要相同优先级按提交顺序处理时，应额外保存序号并作为第二排序条件。已入队对象的排序字段不应随意修改，否则队列不会自动根据变化重新整理。

`PriorityQueue` 不允许 `null`，不适合多个线程直接并发修改；并发需求需选择适用的并发容器并设计业务同步。空队列 `poll` 返回 `null`。

## 手练与答案

加入 2、2、1，逐次 `poll` 得到 1、2、2，但两个相同优先级任务之间的先后不能保证。若要较大数字优先，可传 `Comparator.reverseOrder()`。

参考：[教程：使用PriorityQueue](https://liaoxuefeng.com/books/java/collection/priority-queue/index.html)；[Oracle 文档](https://docs.oracle.com/javase/tutorial/collections/implementations/queue.html)。
