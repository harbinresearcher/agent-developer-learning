# Date和Calendar

> **重要程度：高**｜**手练：需要**
> **重点**：旧类型的可变性、零起始月份、UTC 与互转。

## 为什么仍需了解旧 API

维护旧项目时经常遇到 `java.util.Date` 和 `Calendar`。Date 主要表示时间线上的时刻，显示时格式化规则和时区会影响文本；Calendar 用时区规则操作日期字段。新代码通常优先使用 `java.time`，再在边界与旧 API 互转。

## 完整例子

保存为 `LegacyDateDemo.java`，编译运行；日志写到标准错误。例子固定使用 UTC，避免依赖本机默认时区。

```java
import java.time.Instant;
import java.util.Calendar;
import java.util.Date;
import java.util.TimeZone;
import java.util.logging.Logger;
public class LegacyDateDemo {
    private static final Logger logger = Logger.getLogger(LegacyDateDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：检查旧日期类型
         * ========================================================================
         * 数据源：固定的 epoch 时刻与 UTC 日历
         * 操作：
         * 1) 在 Date 和 Instant 之间转换
         * 2) 显示 Calendar 的月份编码
         */
        logger.info("开始检查旧日期类型");
        // 1.1 由固定时刻构造旧 Date
        Date date = Date.from(Instant.EPOCH);
        // 1.2 设置 UTC 日历并查询月份
        Calendar calendar = Calendar.getInstance(TimeZone.getTimeZone("UTC"));
        calendar.clear();
        calendar.set(2024, Calendar.JANUARY, 2, 10, 0, 0);
        System.out.println("instant=" + date.toInstant());
        System.out.println("monthIndex=" + calendar.get(Calendar.MONTH));
        System.out.println("calendarInstant=" + calendar.toInstant());
        logger.info("旧日期类型检查完成");
    }
}
```

标准输出：

```text
instant=1970-01-01T00:00:00Z
monthIndex=0
calendarInstant=2024-01-02T10:00:00Z
```

## 执行过程

`Date.from(Instant.EPOCH)` 创建 epoch 时刻，`toInstant()` 转回同一时刻。Calendar 指定 UTC 并先 `clear()`，避免残留未设置字段；`set` 使用 `Calendar.JANUARY` 设置一月。`get(Calendar.MONTH)` 返回 0，因为该旧 API 的月份从 0 开始；换成 Instant 后显示 UTC 的 2024-01-02 10:00。

## 常见陷阱

`Date` 和 `Calendar` 是可变对象；跨方法传递时对方修改可能影响原对象，必要时做防御性复制。Date 自身不保存“用户所在地区时区”；不要依赖 `Date.toString()` 的默认时区输出来判断存储的真实时刻。

Calendar 月份从 0 开始，日期字段和星期规则也有各自约定；默认的宽松解析可能调整非法值，严格校验要按 API 配置并测试。`SimpleDateFormat` 可变且不是线程安全的共享格式器；不要在多线程中无保护地共享同一实例。

## 工作中怎么用

使用 `date.toInstant()` 和 `Date.from(instant)` 在边界互转，并明确毫秒精度和时区。日期或本地日程不能在没有业务时区的情况下随意转换成唯一时刻；数据库驱动和协议的具体类型映射需在项目环境验证。

## 手练与答案

`Calendar.JANUARY` 的整数值为 0；二月为 1。把时区改掉而保持同一组本地字段，转换出的 Instant 可能改变，不能把“本地显示相同”当作“同一时刻”。

参考：[廖雪峰：Date和Calendar](https://liaoxuefeng.com/books/java/datetime/date-calendar/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Calendar.html)。
