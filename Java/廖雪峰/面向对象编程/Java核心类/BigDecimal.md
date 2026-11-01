# BigDecimal

> **重要程度：高**｜**手练：需要**
> **重点**：精确十进制构造、scale、舍入和数值比较。

## 为什么需要十进制计算

二进制浮点不能精确表示所有十进制小数，因此计算金额等需要明确十进制精度的值时，通常使用 `BigDecimal`，同时规定币种、小数位和舍入规则。`new BigDecimal("0.10")` 从十进制文本得到明确值；避免 `new BigDecimal(0.1)`，因为传入的 double 已是二进制近似值。`BigDecimal.valueOf(double)` 使用不同的转换方式，但也不能修复此前浮点运算已经造成的误差。

`BigDecimal` 不可变，`add` 等方法返回新对象；若忽略返回值，原对象不会改变。

除法可能出现无限循环小数，项目必须明确精度或小数位和 `RoundingMode`；不要随意吞掉需要舍入的异常。`equals` 同时比较数值和 scale，`compareTo` 按数值比较；例如 `1.0` 与 `1.00` 的数值比较相等，但 `equals` 不一定相等。

手练：计算 `0.1 + 0.2` 与 `1 / 3`，分别明确构造方式和保留两位小数的舍入规则；比较 `equals` 与 `compareTo`。

来源：[教程：BigDecimal](https://liaoxuefeng.com/books/java/oop/core/bigdecimal/index.html)。

## 可运行例子

保存为 `MoneyDemo.java`，执行 `javac -encoding UTF-8 MoneyDemo.java`，再执行 `java MoneyDemo`。

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.logging.Logger;

public class MoneyDemo {
    private static final Logger logger = Logger.getLogger(MoneyDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：计算并比较金额
         * ========================================================================
         * 目标：观察十进制计算和比较方式。
         * 数据源：测试用十进制字符串。
         * 操作：
         * 1) 相加并输出结果。
         * 2) 比较数值和小数位数。
         */
        logger.info("开始计算测试金额");
        // 1.1 从字符串创建数值并相加
        BigDecimal first = new BigDecimal("0.10");
        BigDecimal second = new BigDecimal("0.20");
        BigDecimal total = first.add(second).setScale(2, RoundingMode.HALF_UP);
        // 1.2 输出金额及两种比较结果
        System.out.println(total);
        System.out.println(new BigDecimal("1.0").compareTo(new BigDecimal("1.00")) == 0);
        System.out.println(new BigDecimal("1.0").equals(new BigDecimal("1.00")));
        logger.info("测试金额计算完成");
    }
}
```

标准输出为 `0.30`、`true`、`false`。`0.10` 与 `0.20` 相加后得到 0.30；`setScale(2, RoundingMode.HALF_UP)` 指定两位小数和舍入方式。`1.0` 与 `1.00` 的数值相同，所以 `compareTo` 返回 0；但它们的小数位数（scale）不同，所以 `equals` 为 false。

## 除法与舍入

`new BigDecimal("1").divide(new BigDecimal("3"))` 不能得到有限位十进制精确结果，会抛 `ArithmeticException`。如果业务规定保留两位小数并采用 HALF_UP，可以调用 `divide(new BigDecimal("3"), 2, RoundingMode.HALF_UP)`，结果为 `0.33`。这只是示例规则，不代表所有财务业务都适用 HALF_UP。

`setScale` 返回新值；减小小数位可能需要舍入。`RoundingMode.UNNECESSARY` 表示不允许舍入，需要舍入时会抛异常。运算的每一步都舍入与最后统一舍入可能产生不同结果，应按合同或业务规则确定时机。

## 比较和集合

`compareTo` 按数值比较；`equals` 同时考虑数值和 scale，因此 `1.0` 与 `1.00` 在 `HashMap` 中可能是不同键，而自然排序的 `TreeMap` 用比较结果确定键等价，可能将它们视为同一排序键。做业务去重或存储前要明确规范化规则，不要混用两种相等语义。

## 手练与答案

1. `0.10.add(0.20)` 如果不保存返回值，原来的 `0.10` 仍不变。
2. 1 除以 3，保留两位、HALF_UP：`0.33`；不指定允许舍入的规则会抛 `ArithmeticException`。
3. `1.0` 与 `1.00`：`compareTo` 为 0，但 `equals` 为 false。

参考：[Oracle BigDecimal](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/BigDecimal.html)。
