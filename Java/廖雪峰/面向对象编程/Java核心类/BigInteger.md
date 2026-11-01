# BigInteger

> **重要程度：高**｜**手练：需要**
> **重点**：大整数、不可变性、精确转换。

## 解决什么问题

`long` 的范围有限，超过范围的普通整数运算会溢出。`BigInteger` 表示任意精度整数，适合确实可能超过固定整型范围的整数计算；它不表示小数，十进制金额计算应另看 `BigDecimal`。

## 完整例子

保存为 `BigIntegerDemo.java`，用 `javac -encoding UTF-8 BigIntegerDemo.java` 编译，再运行 `java BigIntegerDemo`。日志写到标准错误。

```java
import java.math.BigInteger;
import java.util.logging.Logger;
public class BigIntegerDemo {
    private static final Logger logger = Logger.getLogger(BigIntegerDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：计算超过 long 范围的整数
         * ========================================================================
         * 数据源：十进制字符串与整数一
         * 操作：
         * 1) 构造大整数并计算新值
         * 2) 输出原值及结果
         */
        logger.info("开始计算大整数");
        // 1.1 用字符串构造并计算
        BigInteger original = new BigInteger("9223372036854775807");
        BigInteger result = original.add(BigInteger.ONE);
        // 1.2 输出两个独立值
        System.out.println("original=" + original);
        System.out.println("result=" + result);
        logger.info("大整数计算完成");
    }
}
```

标准输出：

```text
original=9223372036854775807
result=9223372036854775808
```

## 代码和实践

字符串 `9223372036854775807` 是 `Long.MAX_VALUE`。`add(BigInteger.ONE)` 返回一个新对象，结果为 9223372036854775808，原对象的值不变。`BigInteger` 是不可变的：调用 `add`、`multiply` 等后，如果丢弃返回值，就没有保存新结果。

使用 `new BigInteger("123")` 解析整数字符串；格式非法会抛 `NumberFormatException`。适合从已有 long 构造时可用 `BigInteger.valueOf(value)`。比较大小用 `compareTo`：结果为负、0、正分别表示小于、等于、大于；比较数值相等可用 `equals`。

整数除法 `divide` 舍弃小数部分；除数为零抛 `ArithmeticException`。`longValue()` 等转换可能丢失高位；需要溢出时拒绝转换，用 `longValueExact()`，超出 long 范围会抛 `ArithmeticException`。任意精度不是无限资源：输入长度应受业务限制，极大数字会消耗内存与计算时间。

## 手练与答案

1. 把 `original.add(BigInteger.ONE)` 的结果不赋值，再输出 original：仍为 9223372036854775807。
2. 对 result 调用 `longValueExact()`：抛 `ArithmeticException`，因为它大于 long 最大值。

参考：[Oracle BigInteger](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/BigInteger.html)。

课程原文：[廖雪峰：BigInteger](https://liaoxuefeng.com/books/java/oop/core/biginteger/index.html)。
