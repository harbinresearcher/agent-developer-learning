# ZonedDateTime

> **重要程度：高**｜**手练：需要**
> **重点**：地区时区、同一时刻转换、夏令时。

## 解决什么问题

`ZonedDateTime` 把本地日期时间、地区时区和适用偏移关联起来。它适合描述某个地区的时间，并可转换为时间线上的 `Instant`；同一时刻在不同地区的显示可以不同。

## 完整例子

保存为 `ZoneDemo.java`，编译运行；日志写到标准错误。示例使用固定的 2024 年时刻。

```java
import java.time.*;
import java.util.logging.Logger;

public class ZoneDemo {
    private static final Logger logger = Logger.getLogger(ZoneDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：转换同一时刻的时区
         * ========================================================================
         * 数据源：程序内固定的示例时间
         * 操作：
         * 1) 创建时间对象
         * 2) 转换或计算并检查结果
         */
        logger.info("开始处理时间");
        // 1.1 创建 UTC 时刻和两个地区时区
        Instant instant = Instant.parse("2024-01-01T00:00:00Z");
        ZonedDateTime shanghai = instant.atZone(ZoneId.of("Asia/Shanghai"));
        ZonedDateTime newYork = shanghai.withZoneSameInstant(ZoneId.of("America/New_York"));
        // 1.2 输出表示和时刻相等结果
        System.out.println("shanghai=" + shanghai);
        System.out.println("newYork=" + newYork);
        System.out.println("same=" + shanghai.toInstant().equals(newYork.toInstant()));
        logger.info("时间处理完成");
    }
}
```

标准输出：

```text
shanghai=2024-01-01T08:00+08:00[Asia/Shanghai]
newYork=2023-12-31T19:00-05:00[America/New_York]
same=true
```

## 执行过程

先从 UTC 时刻创建上海时间 2024-01-01 08:00。`withZoneSameInstant` 改变展示时区但保持同一时刻，所以纽约显示上一天 19:00。两者转为 Instant 后相等。

地区时区由规则确定偏移；不能把本例纽约的 `-05:00` 固定当成全年偏移。`withZoneSameInstant` 与保持本地时间的转换语义不同；跨区展示通常要明确要求“时刻不变”还是“钟表显示不变”。

## 夏令时和业务边界

某些地区调钟时，本地时间可能有空缺（不存在）或重叠（出现两次）。`LocalDateTime.atZone` 有默认解析规则：空缺通常向后调整，重叠通常选较早偏移；预约、结算等业务若不能接受默认行为，应检查 `ZoneRules` 的有效偏移并明确拒绝或选择策略。时区规则随地区政策和 JDK 数据更新，业务测试要覆盖目标地区和日期。

跨系统保存时刻时可使用 UTC 或带偏移的标准格式；若业务依赖未来地区日程，还需保留地区 ZoneId 和相关本地意图，固定偏移无法完整描述未来规则。

## 手练与答案

例子中纽约日期是 2023-12-31，上海是 2024-01-01，但 Instant 相同。只比较本地日期和时间不能正确判断跨时区事件是否为同一时刻。

参考：[廖雪峰：ZonedDateTime](https://liaoxuefeng.com/books/java/datetime/zoned-datetime/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/ZonedDateTime.html)。
