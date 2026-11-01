# Java集合简介

> **重要程度：高**｜**手练：需要**
> **重点**：接口与实现、列表、集合、映射。

## 解决什么问题

数组长度固定。集合框架提供可按接口使用的容器和操作，使程序能够存放数量可变的数据。`List`、`Set`、`Queue` 属于 `Collection` 家族；`Map` 是独立的键值映射接口，不继承 `Collection`。

## 完整例子

保存为 `CollectionIntroDemo.java`，用 `javac -encoding UTF-8 CollectionIntroDemo.java` 编译，再运行 `java CollectionIntroDemo`；日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;
public class CollectionIntroDemo {
    private static final Logger logger = Logger.getLogger(CollectionIntroDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：比较三种容器
         * ========================================================================
         * 数据源：两个重复的名字及一个键值对
         * 操作：
         * 1) 分别存入 List、Set 和 Map
         * 2) 输出元素数量和查询结果
         */
        logger.info("开始比较容器");
        // 1.1 创建并填充容器
        List<String> list = new ArrayList<>();
        Set<String> set = new HashSet<>();
        Map<String, Integer> map = new HashMap<>();
        list.add("Ada"); list.add("Ada");
        set.add("Ada"); set.add("Ada");
        map.put("Ada", 2);
        // 1.2 输出容器观察结果
        System.out.println("listSize=" + list.size());
        System.out.println("setSize=" + set.size());
        System.out.println("count=" + map.get("Ada"));
        logger.info("容器比较完成");
    }
}
```

标准输出：

```text
listSize=2
setSize=1
count=2
```

## 执行过程

`List` 保留两个重复元素，所以大小为 2；`Set` 按它的相等规则不重复保存同一元素，所以大小为 1；`Map` 把名字映射到次数，查询 Ada 得到 2。变量声明为接口类型，右侧用具体实现创建对象；这样调用方可以依赖接口契约，但替换实现前仍要检查顺序、空值、性能与并发行为。

## 根据需求选择

- `List`：顺序和位置重要，可以重复。`ArrayList` 适合常见下标读取；插入删除成本取决于位置和实现。
- `Set`：关注元素是否存在、去重。`HashSet` 不保证遍历顺序，是否相等依赖元素契约。
- `Map`：根据键查值。`HashMap` 不保证遍历顺序；有序键和范围查询可看 `TreeMap`。
- `Queue` / `Deque`：需要按队列、双端队列或栈的规则处理元素。实现可能按优先级而非 FIFO。

泛型参数表达元素类型，`List<Integer>` 不能写 `List<int>`；基本类型需要包装类型。`Collection`、`Collections`、`Map` 不是同一个东西。可变容器通常不能假设线程安全。

## 手练与答案

把第二次 `list.add` 删除：List 大小为 1。再 `map.put("Ada", 3)`：同一键的值被替换，`map.get("Ada")` 为 3，键数量仍为 1。

课程原文：[廖雪峰：Java集合简介](https://liaoxuefeng.com/books/java/collection/quick-start/index.html)。
