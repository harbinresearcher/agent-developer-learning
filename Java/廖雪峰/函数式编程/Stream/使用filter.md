# 使用filter

> **重要程度：高**｜**手练：需要**
> **重点**：Predicate、空值、筛选与源集合。

## 解决什么问题

`filter` 保留使 Predicate 返回 true 的元素，其余元素不进入后续流水线。它是中间操作，不会因为创建了筛选条件就立即遍历源数据。

## 完整例子

保存为 `FilterStreamDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.logging.Logger;
public class FilterStreamDemo {
    private static final Logger logger = Logger.getLogger(FilterStreamDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：筛选非空的较长名字
         * ========================================================================
         * 数据源：程序中准备的固定示例数据
         * 操作：
         * 1) 创建函数或数据流
         * 2) 处理输入并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备包含空引用的名字列表
        List<String> names = Arrays.asList("Ada", null, "Bo", "Ming");
        // 1.2 先排除 null，再按长度筛选
        List<String> selected = names.stream()
                .filter(Objects::nonNull)
                .filter(name -> name.length() >= 3)
                .collect(Collectors.toList());
        System.out.println("selected=" + selected);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
selected=[Ada, Ming]
```

## 执行过程

列表包含 Ada、null、Bo、Ming。第一步移除 null，第二步保留长度至少为 3 的字符串；终止操作 collect 触发处理，得到 Ada、Ming。顺序流和有序源在此保留相对遇到顺序。

把两个条件调换会让 null 在调用 `length()` 时触发 `NullPointerException`。如果业务规定 null 是数据错误，静默过滤可能掩盖问题，应选择拒绝或记录安全诊断，而不是机械地使用 `nonNull`。

## 工作中怎么用

Predicate 应清楚表达筛选条件，尽量无副作用，也不要在流处理中修改数据源。多个条件可以拆开提高可读性；`filter` 不修改源列表。检查结果时还应覆盖全不匹配、全部匹配和空集合等边界。

## 手练与答案

把阈值改为 2：结果为 `[Ada, Bo, Ming]`。如果输入为空列表，collect 得到空列表；不会因为没有元素自动抛异常。

参考：[廖雪峰：使用filter](https://liaoxuefeng.com/books/java/functional/stream/filter/index.html)；[Oracle Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
