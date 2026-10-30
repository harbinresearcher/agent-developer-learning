# super 下界通配符

> **重要程度：高**｜**手练：需要**
> **重点**：理解 `List<? super Integer>`；能向集合写入 Integer；解释为什么读取只能安全地按 Object 使用。

## 先看问题

要写一个方法，往集合里放整数。如果参数写成 `List<Integer>`，调用方只能传整数列表；但一个 `List<Number>` 或 `List<Object>` 也能保存整数。

`List<? super Integer>` 表示：列表的元素类型是 `Integer` 或它的某个父类型。方法可以向其中写入 `Integer`；因为不论具体是哪一种候选类型，整数都符合下界约束。

## 完整例子

保存为 `WildcardDemo.java`，在文件目录运行 `javac -encoding UTF-8 WildcardDemo.java`，再运行 `java WildcardDemo`。

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.logging.Logger;

public class WildcardDemo {
    private static final Logger logger = Logger.getLogger(WildcardDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：读取数字并写入整数
         * ========================================================================
         * 目标：区分 extends 读取和 super 写入。
         * 数据源：整数列表和 Number 列表。
         * 操作：
         * 1) 从整数列表读取并求和。
         * 2) 向 Number 列表写入整数。
         */
        logger.info("开始演示通配符");
        // 1.1 创建数据并读取数字
        List<Integer> values = Arrays.asList(1, 2, 3);
        System.out.println("总和: " + sum(values));
        // 1.2 向 Number 列表添加整数
        List<Number> output = new ArrayList<>();
        addOne(output);
        System.out.println("写入结果: " + output);
        logger.info("通配符演示完成");
    }

    static double sum(List<? extends Number> values) {
        double result = 0;
        for (Number value : values) {
            result += value.doubleValue();
        }
        return result;
    }

    static void addOne(List<? super Integer> values) {
        values.add(1);
    }
}
```

标准输出依次为 `总和: 6.0` 和 `写入结果: [1]`。`sum` 用上界通配符读取，`addOne` 用下界通配符写入。

### 先看 `sum`

`sum(List<? extends Number> values)` 接收整数列表，按 `Number` 逐项读取并求和。它不知道列表实际参数类型，因此不能随意放入普通数字。详见 [[extends通配符]]。

### 再看 `addOne`

`addOne(List<? super Integer> values)` 接收 `List<Number>`，再向它添加整数 `1`。同样，该参数也可接收 `List<Integer>` 或 `List<Object>`。

读取时要小心：如果传入的是 `List<Object>`，里面可能还有字符串等其他对象。因此方法从这个参数读取的元素，通常只能安全地按 `Object` 使用；不能未经检查就当成 `Integer`。

## 怎么记

先看数据流：**从参数集合取 Number**，考虑 `extends Number`；**向参数集合放 Integer**，考虑 `super Integer`。口诀“生产者 extends、消费者 super”是辅助，不取代对读取类型和写入类型的判断。

## 手练与自测

1. 把示例中的 `output` 改成 `List<Object>`，`addOne(output)` 还可以编译吗？
2. 为什么不能把下界通配符列表里取出的元素直接赋给 `Integer` 变量？
3. `List<Integer>` 能否传给 `List<? super Integer>`？`List<Double>` 呢？

**答案要点**：① 可以，Object 是 Integer 的父类型。② 实际列表可能是 `List<Object>`，里面还有其他类型。③ 前者可以；Double 不是 Integer 的父类型，后者不可以。

参考：[廖雪峰：super通配符](https://liaoxuefeng.com/books/java/generics/super/index.html)；[Oracle：下界通配符](https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html)。
