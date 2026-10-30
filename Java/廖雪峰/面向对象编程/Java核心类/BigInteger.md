# BigInteger

> **重要程度：中**｜**手练：建议**
> **重点**：超大整数、不变对象、算术方法和边界。

`BigInteger` 表示任意精度整数，适合数值超出 `long` 范围的场景。它是不可变对象，算术方法如 `add`、`multiply` 返回新结果；不能用普通 `+` 直接做 `BigInteger` 加法。

转换回固定宽度整数可能丢失信息。需要检测越界时应选择带精确检查的方法并处理异常；大整数运算仍受到内存和时间限制，并非无限成本。

手练：计算超过 `long` 范围的整数之和；尝试转换为 `long` 并检查越界行为。

来源：[教程：BigInteger](https://liaoxuefeng.com/books/java/oop/core/biginteger/index.html)。
