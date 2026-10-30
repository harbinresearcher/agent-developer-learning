# 使用 List

> **重要程度：高**｜**手练：需要**
> **重点**：按顺序保存元素；使用 `add`、`get`、`remove`；理解下标、重复值和具体实现的区别。

## 先解决什么问题

数组长度固定。如果要动态增加学生姓名，可以用 `List<String>`。List 按顺序组织元素，允许出现相同值；每个位置有从 0 开始的下标。`List` 是接口，`ArrayList` 是常见实现。

## 完整例子

保存为 `ListDemo.java`；在该目录运行 `javac -encoding UTF-8 ListDemo.java`，再运行 `java ListDemo`。
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.logging.Logger;

public class ListDemo {
    private static final Logger logger = Logger.getLogger(ListDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：读取并修改学生列表
         * ========================================================================
         * 目标：观察顺序、重复元素和索引变化。
         * 数据源：三条姓名。
         * 操作：
         * 1) 创建可修改的列表并读取元素。
         * 2) 删除首项后检查剩余列表。
         */
        logger.info("开始处理学生列表");
        // 1.1 创建列表并读取首项
        List<String> names = new ArrayList<>(Arrays.asList("小明", "小红", "小明"));
        System.out.println("数量: " + names.size());
        System.out.println("首项: " + names.get(0));
        // 1.2 删除下标为 0 的元素并查看结果
        names.remove(0);
        System.out.println("删除后: " + names);
        logger.info("学生列表处理完成");
    }
}
```

标准输出依次是 `数量: 3`、`首项: 小明`、`删除后: [小红, 小明]`。

### 发生了什么

1. `new ArrayList<>(...)` 创建可修改列表，保存三个名字；两次“小明”都保留。
2. `size()` 返回元素数量；`get(0)` 读取下标 0，即第一个元素。
3. `remove(0)` 删除第一个元素。后面的元素向前移动，原来下标 1 的“小红”现在位于下标 0。

## 实际使用要分清

`get(size())` 越界，因为最后一个合法下标是 `size() - 1`。`add(value)` 通常在末尾追加；按指定位置插入会影响后面的下标。遍历期间删除元素，要遵守迭代器和集合的修改规则，不能盲目在增强 for 中直接结构性修改同一个列表。

`List<Integer>` 有个易错点：`remove(1)` 调用的是按**下标**删除；如果要按值删除整数 1，应明确选择接受对象的重载，例如 `remove(Integer.valueOf(1))`。

`List.of(...)` 创建的列表不支持添加、删除等结构性修改，也不接受 null；不要因为接收类型是 `List`，就认为所有 List 对象都可以修改。

## 手练与自测

1. 把首项删除后，再读取下标 0，会得到谁？
2. 设列表为 `[10, 20, 30]`，`remove(1)` 删除什么？
3. `List.of("A", "B")` 上调用 `add("C")` 会怎样？

**答案要点**：① 小红。② 下标 1 对应的 20。③ 抛 `UnsupportedOperationException`。

参考：[教程：使用List](https://liaoxuefeng.com/books/java/collection/list/index.html)；[Java List API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html)。
