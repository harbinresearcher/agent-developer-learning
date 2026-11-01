# 使用TreeMap

> **重要程度：高**｜**手练：需要**
> **重点**：键排序、比较器、范围查询。

## 解决什么问题

需要按键排序、查找相邻键或取键范围时，可用 `TreeMap`。它是 `NavigableMap` 的实现，根据自然顺序或构造时给出的比较器排列键；与 `HashMap` 不承诺键顺序的行为不同。

## 完整例子

保存为 `TreeMapDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.logging.Logger;

public class TreeMapDemo {
    private static final Logger logger = Logger.getLogger(TreeMapDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：按键排序和查询
         * ========================================================================
         * 数据源：程序中准备的示例数据
         * 操作：
         * 1) 创建集合并填入数据
         * 2) 查询并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备按键自然顺序排列的映射
        NavigableMap<Integer, String> map = new TreeMap<>();
        map.put(20, "B");
        map.put(10, "A");
        map.put(30, "C");
        // 1.2 按键顺序输出并查找临近键
        System.out.println("keys=" + map.keySet());
        System.out.println("floor25=" + map.floorKey(25));
        System.out.println("range=" + map.subMap(10, true, 30, false));
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
keys=[10, 20, 30]
floor25=20
range={10=A, 20=B}
```

## 执行过程

按 20、10、30 的顺序插入，但 `keySet()` 按键从小到大显示 `[10, 20, 30]`。`floorKey(25)` 返回不大于 25 的最大键 20。`subMap(10, true, 30, false)` 返回键在 `[10, 30)` 的范围视图，包含 10 和 20，不包含 30。

## 容易出错的边界

`subMap` 通常是原映射的视图，不是独立副本；通过视图进行允许的修改会反映到原映射，并受范围限制。需要独立结构时可复制，例如 `new TreeMap<>(view)`。不存在符合条件的 `floorKey` 时会返回 null；应根据 API 分辨“没有键”和存储了空值。

TreeMap 用比较结果判断键的顺序和等价：比较器返回 0 时视为同一排序键，后一次 `put` 会替换对应的值。比较器若与 `equals` 不一致，可能不符合普通 `Map` 契约的预期。自然顺序默认不接受 null 键；自定义比较器的空键行为看其规则。

## 工作中怎么选

需要有序范围或上下界查询时选 TreeMap；只需普通键查找时结合实际使用、复杂度和性能测量选择 HashMap。TreeMap 不提供并发修改安全保证。

## 手练与答案

`floorKey(5)` 返回 null；`subMap(10, true, 30, true)` 包含 10、20、30。向 `map.put(20, "新值")` 后，键数量仍为 3，20 对应值被替换。

参考：[教程：使用TreeMap](https://liaoxuefeng.com/books/java/collection/tree-map/index.html)；[Oracle 文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeMap.html)。
