# DateTimeFormatter

> **重要程度：高**｜**手练：需要**
> **重点**：格式化、严格解析、年份与时区。

## 解决什么问题

`DateTimeFormatter` 在日期时间对象和文本之间转换。模式、区域、时区与解析严格程度共同决定结果；接口协议应明确格式，而不是依赖运行机器的默认设置。

## 完整例子

保存为 `FormatterDemo.java`，编译运行；日志写到标准错误。

```java
import java.time.*;
import java.util.logging.Logger;
import java.time.format.DateTimeFormatter;
import java.time.format.ResolverStyle;
import java.time.format.DateTimeParseException;
import java.util.Locale;

public class FormatterDemo {
    private static final Logger logger = Logger.getLogger(FormatterDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：格式化并严格解析日期
         * ========================================================================
         * 数据源：程序内固定的示例时间
         * 操作：
         * 1) 创建时间对象
         * 2) 转换或计算并检查结果
         */
        logger.info("开始处理时间");
        // 1.1 明确模式、区域与严格解析
        DateTimeFormatter format = DateTimeFormatter.ofPattern("uuuu-MM-dd", Locale.ROOT)
                .withResolverStyle(ResolverStyle.STRICT);
        LocalDate date = LocalDate.of(2024, 2, 29);
        // 1.2 格式化并演示无效日期
        System.out.println("text=" + format.format(date));
        try {
            LocalDate.parse("2023-02-29", format);
        } catch (DateTimeParseException ex) {
            System.out.println("invalid date");
        }
        logger.info("时间处理完成");
    }
}
```

标准输出：

```text
text=2024-02-29
invalid date
```

## 执行过程

模式 `uuuu-MM-dd` 表示年份、两位月份、两位日；示例指定 Locale.ROOT 和 STRICT。2024 年是闰年，2 月 29 日格式化成功。2023 年没有 2 月 29 日，严格解析抛 `DateTimeParseException`，由 catch 输出错误。

## 容易混淆的格式符号

`MM` 是月份，`mm` 是分钟；`HH` 是 24 小时制小时。`u` 是 proleptic year，`y` 是年号内年份，`Y` 是 week-based-year（周基年）；跨年附近 `YYYY` 可能与日历年不同，不要把它当普通年份。模式语义和严格解析需配合完整字段考虑；给用户显示或接收时间时还要明确 Locale。

`LocalDate` 和 `LocalDateTime` 没有时区。带 `Z` 或偏移的文本应按其语义解析为相应类型，不能仅去掉偏移就假装保留了时刻。`DateTimeFormatter` 不可变且线程安全；这与旧 `SimpleDateFormat` 的并发行为不同。

## 手练与答案

`2024-02-29` 在本例可严格解析；`2023-02-29` 抛 `DateTimeParseException`。把模式误写为 `uuuu-mm-dd`：mm 代表分钟而非月份，不能得到预期的日期格式。

参考：[廖雪峰：DateTimeFormatter](https://liaoxuefeng.com/books/java/datetime/datetime-formatter/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/format/DateTimeFormatter.html)。
