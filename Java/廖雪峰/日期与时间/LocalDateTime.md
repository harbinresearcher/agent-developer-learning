# LocalDateTime

> **重要程度：高**｜**手练：需要**
> **重点**：无时区、不可变、日历运算和边界。

## 解决什么问题

`LocalDateTime` 表示本地日历日期和钟表时间，没有时区或偏移。它适合确实只关心本地显示的时间；不能单独用于确定跨地区事件发生的唯一时刻。只有日期时用 `LocalDate`，只有时间时用 `LocalTime`。

## 完整例子

保存为 `LocalTimeDemo.java`，编译运行；日志写到标准错误。

```java
import java.time.*;
import java.util.logging.Logger;

public class LocalTimeDemo {
    private static final Logger logger = Logger.getLogger(LocalTimeDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：处理本地日期时间
         * ========================================================================
         * 数据源：程序内固定的示例时间
         * 操作：
         * 1) 创建时间对象
         * 2) 转换或计算并检查结果
         */
        logger.info("开始处理时间");
        // 1.1 创建本地时间并计算下一月
        LocalDateTime first = LocalDateTime.of(2024, 1, 31, 10, 30);
        LocalDateTime next = first.plusMonths(1);
        // 1.2 展示原值、新值与日期
        System.out.println("first=" + first);
        System.out.println("next=" + next);
        System.out.println("date=" + next.toLocalDate());
        logger.info("时间处理完成");
    }
}
```

标准输出：

```text
first=2024-01-31T10:30
next=2024-02-29T10:30
date=2024-02-29
```

## 执行过程

初始值是 2024 年 1 月 31 日 10:30。`plusMonths(1)` 返回新对象；由于 2024 年 2 月没有 31 日，结果调整到该月最后一天 2 月 29 日。原对象仍是 1 月 31 日，体现 java.time 类型的不可变性。`toLocalDate()` 只取日期部分。

## 使用边界

`LocalDateTime.now()` 使用某个时钟和时区获得本地值，默认重载依赖系统默认时区；跨环境测试不应假定机器时区一致。`of` 对非法日期通常抛 `DateTimeException`。加月与加固定天数是不同业务规则，月底、闰年需要测试。

将本地时间与 `ZoneId` 结合才能得到地区时区时间；夏令时切换可能产生不存在或重复的本地时间。默认解析如何处理应依据 API 和业务约定，不能假设每个本地值都唯一对应时刻。

## 手练与答案

将年份改为 2023：加一个月后为 2023-02-28 10:30。只改变 `next` 不会修改 `first`。判断事件先后若涉及不同时区，应先明确时区并转成可比较的时刻。

参考：[廖雪峰：LocalDateTime](https://liaoxuefeng.com/books/java/datetime/local-datetime/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/LocalDateTime.html)。
