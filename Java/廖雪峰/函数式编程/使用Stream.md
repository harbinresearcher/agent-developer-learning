# 使用Stream

> **重要程度：高**｜**手练：需要**
> **重点**：数据源、中间操作、终止操作与惰性。

## 解决什么问题

Stream 用流水线表达对数据的聚合处理：数据源 → 零个或多个中间操作 → 终止操作。它不等于集合本身，也不自动保证比循环更快。

## 完整例子

保存为 `StreamPipelineDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.logging.Logger;
public class StreamPipelineDemo {
    private static final Logger logger = Logger.getLogger(StreamPipelineDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：构建成绩处理流水线
         * ========================================================================
         * 数据源：程序中准备的固定示例数据
         * 操作：
         * 1) 创建函数或数据流
         * 2) 处理输入并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备成绩
        List<Integer> scores = Arrays.asList(50, 60, 80);
        // 1.2 筛选及格成绩并转换后收集
        List<Integer> result = scores.stream()
                .filter(score -> score >= 60)
                .map(score -> score * 2)
                .collect(Collectors.toList());
        System.out.println("result=" + result);
        System.out.println("source=" + scores);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
result=[120, 160]
source=[50, 60, 80]
```

## 执行过程

数据源是 `[50, 60, 80]`。`filter` 保留 60 和 80，`map` 将各值乘 2，`collect` 收集成列表 `[120, 160]`。示例的操作没有修改源列表，所以它仍为 `[50, 60, 80]`。

中间操作通常是惰性的：构造 `filter` 和 `map` 时未必开始遍历；终止操作触发计算。实现允许在不影响结果时优化掉部分阶段，因此不能依赖 `peek` 或中间操作副作用一定执行。消费后的 Stream 不应重用；需要第二次处理时从数据源创建新流。

## 工作中怎么用

保持行为函数尽量无状态、不修改正在遍历的数据源。短路操作、并行执行和底层优化会影响调用次数与顺序，不能用副作用推断全部元素必然经过。集合产生的流一般无须单独关闭，但 `Files.lines` 等持有 I/O 资源的流需要按契约关闭。

遇到多层嵌套、复杂异常或需要明确中断的逻辑，普通循环可能更易读。并行流不保证加速，先测量，并考虑共享状态与线程池。

## 手练与答案

阈值改为 80：结果为 `[160]`。删除 collect 并不消费流：流水线通常不会执行；不要用中间步骤中的打印来保证业务动作发生。

参考：[廖雪峰：使用Stream](https://liaoxuefeng.com/books/java/functional/stream/index.html)；[Oracle Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
