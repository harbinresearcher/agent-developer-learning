# 使用Collections

> **重要程度：高**｜**手练：需要**
> **重点**：排序、只读视图和独立副本。

## 解决什么问题

`Collections` 是提供集合操作的工具类；它不是表示一组元素的 `Collection` 接口。它可以排序、反转、创建只读包装等。很多操作会改变传入的原集合，使用前先确认契约。

## 完整例子

保存为 `CollectionsDemo.java`，用 `javac -encoding UTF-8 CollectionsDemo.java` 编译，再运行 `java CollectionsDemo`。日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;
public class CollectionsDemo {
    private static final Logger logger = Logger.getLogger(CollectionsDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：排序并比较集合视图
         * ========================================================================
         * 数据源：一个可修改的整数列表
         * 操作：
         * 1) 排序并创建只读视图与副本
         * 2) 修改原列表后观察差异
         */
        logger.info("开始处理列表");
        // 1.1 排序、创建视图和副本
        List<Integer> values = new ArrayList<>(Arrays.asList(3, 1, 2));
        Collections.sort(values);
        List<Integer> view = Collections.unmodifiableList(values);
        List<Integer> copy = new ArrayList<>(values);
        // 1.2 修改原列表并观察两个结果
        values.add(4);
        System.out.println("view=" + view);
        System.out.println("copy=" + copy);
        logger.info("列表处理完成");
    }
}
```

标准输出：

```text
view=[1, 2, 3, 4]
copy=[1, 2, 3]
```

## 代码和实践

`Collections.sort(values)` 原地把列表排成 1、2、3。`unmodifiableList(values)` 返回只读视图，阻止调用者通过 `view` 修改列表，但它仍观察原列表。`new ArrayList<>(values)` 创建独立的列表结构。原列表追加 4 后，视图显示 4，副本不显示。

通过 `view.add(5)` 会抛 `UnsupportedOperationException`；这不代表其他引用不能修改原列表，也不代表其中的可变元素变成不可变。副本只复制元素引用：如果元素本身可变，修改同一个元素仍可能被两边观察到。

`Collections.reverse(list)` 原地反转；`Collections.sort` 和 `List.sort` 根据自然顺序或比较器排序。先确认列表支持修改，否则排序可能抛异常。Java 10 起 `List.copyOf` 可构造不可修改列表，但仍是浅层语义且不接受 null 元素；项目需核对 JDK。

同步包装方法只提供其文档规定的同步行为，迭代或复合操作仍要遵守对应的同步要求。不要把只读视图误当成线程安全快照。

## 手练与答案

删除 `values.add(4)`，view 与 copy 都显示 `[1, 2, 3]`。改为调用 `view.add(4)`，程序在该处抛 `UnsupportedOperationException`。

课程原文：[廖雪峰：使用Collections](https://liaoxuefeng.com/books/java/collection/collections/index.html)。
