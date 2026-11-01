# 使用reduce

> **重要程度：高**｜**手练：需要**
> **重点**：单位元、结合性、空流与溢出。

## 解决什么问题

`reduce` 把一组元素按给定规则归约为一个结果，例如求和。带单位元的重载在空流时返回单位元；没有单位元的重载返回 Optional，用于表达可能没有元素。

## 完整例子

保存为 `ReduceDemo.java`，用 Java 11 或更高版本编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.logging.Logger;
public class ReduceDemo {
    private static final Logger logger = Logger.getLogger(ReduceDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：归约数字序列
         * ========================================================================
         * 数据源：程序中准备的固定示例数据
         * 操作：
         * 1) 创建流并配置操作
         * 2) 执行终止操作并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备一组数字和一个空流
        List<Integer> values = Arrays.asList(1, 2, 3);
        // 1.2 使用单位元归约和处理空流
        int sum = values.stream().reduce(0, Integer::sum);
        Optional<Integer> empty = Stream.<Integer>empty().reduce(Integer::sum);
        System.out.println("sum=" + sum);
        System.out.println("empty=" + empty.isEmpty());
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
sum=6
empty=true
```

## 执行过程

从单位元 0 开始依次与 1、2、3 相加，得到 6。第二条对空流使用没有单位元的重载，得到空 Optional，`isEmpty()` 为 true；它不是数值 0。

## 单位元和结合性

单位元应满足对操作的单位性质，例如整数加法的单位元是 0；如果错误地用 10，顺序流的本例得到 16，且在并行归约中语义会更复杂。归约函数应符合 API 要求的结合性与无状态等约束；减法一般不满足结合性，不能安全地当作可并行组合的归约规则。

`int` 加法可能溢出，浮点加法受舍入影响也不满足严格数学结合性。金额计算需选择合适类型和舍入规则；大型数据聚合还需考虑数值范围。

## 工作中怎么用

常见数字求和可用 `mapToInt(...).sum()` 等专用操作；复杂可变结果通常用 `collect` 更合适。不要用共享可变列表在并行 reduce 中累积数据。

## 手练与答案

把列表改为空：带单位元的 sum 为 0，无单位元结果仍为空 Optional。单位元改成 10 且列表不变：本例顺序流 sum 为 16，但 10 不是加法单位元。

参考：[廖雪峰：使用reduce](https://liaoxuefeng.com/books/java/functional/stream/reduce/index.html)；[Oracle Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
