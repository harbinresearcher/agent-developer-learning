# 创建Stream

> **重要程度：高**｜**手练：需要**
> **重点**：集合、数组、生成器、有界与无限流。

## 解决什么问题

Stream 可以来自集合、数组、固定元素或生成函数。创建方式决定元素类型、是否有顺序、是否需要管理底层资源；它通常只描述来源，终止操作才启动处理。

## 完整例子

保存为 `CreateStreamDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.logging.Logger;
public class CreateStreamDemo {
    private static final Logger logger = Logger.getLogger(CreateStreamDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：从不同数据源创建流
         * ========================================================================
         * 数据源：程序中准备的固定示例数据
         * 操作：
         * 1) 创建函数或数据流
         * 2) 处理输入并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 从数组创建基本类型流
        int[] values = {1, 2, 3};
        int sum = Arrays.stream(values).sum();
        // 1.2 创建无限整数序列并限制数量
        List<Integer> first = Stream.iterate(1, n -> n + 1)
                .limit(3)
                .collect(Collectors.toList());
        System.out.println("sum=" + sum);
        System.out.println("first=" + first);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
sum=6
first=[1, 2, 3]
```

## 执行过程

`Arrays.stream(values)` 为 int 数组创建 `IntStream`，`sum()` 得到 6。`Stream.iterate(1, n -> n + 1)` 描述从 1 开始递增的无限序列；`limit(3)` 只取前三项，`collect` 得到 `[1, 2, 3]`。

集合可用 `collection.stream()`，固定对象可用 `Stream.of(...)`。注意 `Stream.of(intArray)` 与 `Arrays.stream(intArray)` 类型不同：前者可产生包含一个 int 数组的对象流，后者产生 IntStream；不要只看输出就假定元素是整数。

无限流若没有合适的终止边界，收集等操作可能不会结束或耗尽资源。`Files.lines` 等资源型数据源需要关闭；集合和数组流通常不需要额外关闭。已消费或关闭的流不应再次使用。

## 手练与答案

把 limit 改成 5：得到 `[1, 2, 3, 4, 5]`。`Arrays.stream(new int[]{1, 2, 3}).count()` 是 3；`Stream.of(intArray).count()` 在该调用形式下是 1。

参考：[廖雪峰：创建Stream](https://liaoxuefeng.com/books/java/functional/stream/create/index.html)；[Oracle Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
