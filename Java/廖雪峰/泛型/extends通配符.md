# extends 上界通配符

> **重要程度：高**｜**手练：需要**
> **重点**：理解 `List<? extends Number>`；能从中按 `Number` 读取；解释为什么不能随意写入。

## 先看问题

有一个求和方法，希望既能接收 `List<Integer>`，也能接收 `List<Double>`。直接把参数写成 `List<Number>` 不行：虽然 `Integer` 是 `Number` 的子类，`List<Integer>` 却不是 `List<Number>` 的子类型。

参数写成 `List<? extends Number>`，意思是：接收元素类型为 **Number 或某个 Number 子类型**的列表；具体是哪一种，方法内部并不知道。

## 完整例子

保存为 `NumbersDemo.java`。在文件目录运行 `javac -encoding UTF-8 NumbersDemo.java`，再运行 `java NumbersDemo`。

```java
import java.util.Arrays;
import java.util.List;
import java.util.logging.Logger;

public class NumbersDemo {
    private static final Logger logger = Logger.getLogger(NumbersDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：读取不同数字列表
         * ========================================================================
         * 目标：理解上界通配符的读取能力。
         * 数据源：整数列表和浮点数列表。
         * 操作：
         * 1) 调用同一个求和方法。
         * 2) 输出两组结果。
         */
        logger.info("开始计算数字总和");
        // 1.1 创建两种数字列表
        List<Integer> integers = Arrays.asList(1, 2, 3);
        List<Double> decimals = Arrays.asList(1.5, 2.5);
        // 1.2 读取并输出结果
        System.out.println(total(integers));
        System.out.println(total(decimals));
        logger.info("数字总和计算完成");
    }

    static double total(List<? extends Number> values) {
        double result = 0;
        for (Number value : values) {
            result += value.doubleValue();
        }
        return result;
    }
}
```

标准输出依次为 `6.0` 和 `4.0`。本例使用 `double` 方便说明类型关系，不作为精确金额计算的模板。

### 按调用顺序理解

1. `integers` 的类型是 `List<Integer>`，`decimals` 是 `List<Double>`。
2. 两者都能传给 `total(List<? extends Number> values)`，因为元素类型都在 `Number` 的上界范围内。
3. 循环每次从列表读一个元素。虽然不知道具体是 `Integer` 还是 `Double`，但知道它至少可以按 `Number` 使用，所以可以调用 `doubleValue()`。
4. `total` 累加并返回结果。示例没有修改传入列表。

## 为什么不能随便 add

设想 `values` 实际引用的是 `List<Integer>`。如果方法允许向它放入 `Double`，调用方原本的整数列表就会混入错误类型。由于方法内部不知道 `?` 代表哪一种具体子类型，编译器不能安全地接受普通的 `Integer`、`Double` 或 `Number` 写入。

`null` 是特殊情况；而且上界通配符并不代表集合完全不可变，某些修改操作仍可能存在。不要把它简化成“绝对只读集合”。

## 工作中怎么选

方法主要**从集合取值**，且只需要按 `Number` 使用时，上界通配符通常合适。需要往集合放整数时，看 [[super通配符]]。涉及金额时使用合适的十进制类型，并自行定义精度和舍入规则。

## 手练与自测

1. 给 `total` 传入 `List<Integer>` 和 `List<Double>`，为什么两种都能读取为 `Number`？
2. 在 `total` 里尝试 `values.add(1)`。为什么编译器拒绝？
3. `List<Number>` 与 `List<? extends Number>` 的接收范围有什么不同？

**答案要点**：① 两种元素类型都属于 Number 子类型。② `?` 可能是 Double 等其他具体类型，无法保证整数可写入。③ 前者是精确的 Number 参数化类型；后者还可接收 Number 的不同子类型列表。

参考：[廖雪峰：extends通配符](https://liaoxuefeng.com/books/java/generics/extends/index.html)；[Oracle：上界通配符](https://docs.oracle.com/javase/tutorial/java/generics/upperBounded.html)。
