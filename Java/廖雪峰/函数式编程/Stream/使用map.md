# 使用map

> **重要程度：高**｜**手练：需要**
> **重点**：一对一转换、flatMap、null 与副作用。

## 解决什么问题

`map` 把流中的每个元素转换成另一个值，元素类型可以改变；`flatMap` 可把每个元素映射成一个流，再把这些流的元素展开。两者都是中间操作。

## 完整例子

保存为 `MapStreamDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.logging.Logger;
public class MapStreamDemo {
    private static final Logger logger = Logger.getLogger(MapStreamDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：转换及展开数据
         * ========================================================================
         * 数据源：程序中准备的固定示例数据
         * 操作：
         * 1) 创建流并配置操作
         * 2) 执行终止操作并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备分组名字
        List<List<String>> groups = Arrays.asList(
                Arrays.asList("Ada", "Bo"), Arrays.asList("Li"));
        // 1.2 展开每组，再把名字转换为长度
        List<Integer> lengths = groups.stream()
                .flatMap(List::stream)
                .map(String::length)
                .collect(Collectors.toList());
        System.out.println("lengths=" + lengths);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
lengths=[3, 2, 2]
```

## 执行过程

源列表包含两个分组 `[Ada, Bo]` 和 `[Li]`。`flatMap(List::stream)` 将分组展开成 Ada、Bo、Li；`map(String::length)` 将每个名字转换成 UTF-16 单元数，得到 3、2、2。collect 触发处理并收集结果。

若只用 `map(List::stream)`，得到的是流的流，而不是已展开的字符串流。`map` 不会自动删除 null；`String::length` 遇到 null 会抛异常。是否过滤或拒绝空值是业务规则。

## 工作中怎么用

转换函数尽量无副作用，不要在流水线中依赖执行次数、执行顺序或修改源集合。`mapToInt` 等基本类型流可用于数值聚合并减少显式装箱，但性能仍需测量。字符串长度的业务意义也要明确：`length()` 不一定等于码点或用户感知字符数。

## 手练与答案

第二组改为 `Arrays.asList("Li", "Ming")`：结果为 `[3, 2, 2, 4]`。将某个名字改成 null：示例在计算长度时抛 `NullPointerException`。

参考：[廖雪峰：使用map](https://liaoxuefeng.com/books/java/functional/stream/map/index.html)；[Oracle Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
