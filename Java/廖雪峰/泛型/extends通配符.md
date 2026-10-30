# extends通配符

> **重要程度：高**｜**手练：需要**
> **重点**：上界通配符、读取和写入限制。

## 核心概念

`? extends Number` 表示某个未知的 Number 子类型。方法需要从集合读取数字时，这样的参数能接受整数列表、浮点数列表等；读出的元素可以按 Number 使用。

## 工作中注意

由于具体子类型未知，不能安全地往该列表添加普通的 Number、Integer 或 Double 元素；`null` 是特殊情况。上界通配符并不代表集合绝对不可变。

## 手练

写一个从数字列表读取并求和的方法，分别传入整数和浮点数列表；尝试添加整数，观察编译错误。

来源：[教程：extends通配符](https://liaoxuefeng.com/books/java/generics/extends/index.html)。

## 可运行例子

保存为 `NumbersDemo.java`，用 `javac -encoding UTF-8 NumbersDemo.java` 编译，再运行 `java NumbersDemo`。

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

两次输出分别为 `6.0` 和 `4.0`。`total` 只要求能按 `Number` 读取元素，因此两个列表都能传入。例子使用 `double` 演示泛型，不适合作为金额精确计算模板。
