# BigDecimal

> **重要程度：高**｜**手练：需要**
> **重点**：精确十进制构造、scale、舍入和数值比较。

金额等需要十进制精度的计算通常使用 `BigDecimal`。可从十进制字符串构造，避免先把不能精确表示的二进制浮点结果传进去。它不可变，`add` 等方法返回新对象。

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

输出依次为 `0.30`、`true`、`false`。`compareTo` 在这里比较数值；`equals` 还考虑 scale。真实金额计算应按业务明确精度和舍入规则。
