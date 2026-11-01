# 使用EnumMap

> **重要程度：高**｜**手练：需要**
> **重点**：枚举作为键、顺序、缺失值。

## 解决什么问题

当键只能取某个枚举中的固定值，例如订单状态，用 `EnumMap` 能把键类型和允许的取值写清楚。构造时提供枚举的 `Class`，例如 `new EnumMap<>(Status.class)`。

## 完整例子

保存为 `EnumMapDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;

public class EnumMapDemo {
    private static final Logger logger = Logger.getLogger(EnumMapDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：统计枚举状态
         * ========================================================================
         * 数据源：程序中准备的示例数据
         * 操作：
         * 1) 创建集合并填入数据
         * 2) 查询并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 用订单状态作为键
        Map<Status, Integer> counts = new EnumMap<>(Status.class);
        counts.put(Status.DONE, 2);
        counts.put(Status.NEW, 3);
        // 1.2 读取已有值、缺失值和键顺序
        System.out.println("new=" + counts.get(Status.NEW));
        System.out.println("missing=" + counts.get(Status.CANCELLED));
        System.out.println("keys=" + counts.keySet());
        logger.info("示例处理完成");
    }
}

enum Status { NEW, DONE, CANCELLED }
```

标准输出：

```text
new=3
missing=null
keys=[NEW, DONE]
```

## 执行过程

`Status` 定义 NEW、DONE、CANCELLED 三种值。依次插入 DONE 和 NEW 后，查 NEW 得到 3；未插入的 CANCELLED 返回 null。键集按枚举声明顺序输出 `[NEW, DONE]`，不是插入顺序。

## 必须分清的事

`get` 返回 null 可能表示没有这个键，也可能表示该键被显式映射为 null，因为 EnumMap 允许 null 值。需要区分时用 `containsKey`。EnumMap 不允许 null 键；使用其他枚举类型作为键不符合它的键类型约束。

状态枚举更适合表达固定领域概念；若状态来自不受控的外部字符串，先校验并转换，不能假设任何输入都是合法枚举。EnumMap 本身不保证并发修改安全。它的迭代顺序依赖枚举声明顺序；不要把 `ordinal()` 当稳定的外部数据库或协议编码。

## 手练与答案

加入 `counts.put(Status.CANCELLED, null)` 后，`get(CANCELLED)` 仍为 null，但 `containsKey(CANCELLED)` 为 true。改动声明顺序会改变迭代顺序，不应因此改变已经对外约定的状态编码。

参考：[教程：使用EnumMap](https://liaoxuefeng.com/books/java/collection/enum-map/index.html)；[Oracle 文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/EnumMap.html)。
