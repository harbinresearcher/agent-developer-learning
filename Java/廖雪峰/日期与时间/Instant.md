# Instant

> **重要程度：高**｜**手练：需要**
> **重点**：时间线时刻、秒与毫秒、精度。

## 解决什么问题

`Instant` 表示时间线上的时刻，不绑定某个地区的钟表显示。它以秒和纳秒部分表达时间；与地区时间互转时需要指定时区。常用在事件发生时间和跨系统时间点传递中。

## 完整例子

保存为 `InstantDemo.java`，编译运行；日志写到标准错误。

```java
import java.time.*;
import java.util.logging.Logger;

public class InstantDemo {
    private static final Logger logger = Logger.getLogger(InstantDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：转换时间戳单位
         * ========================================================================
         * 数据源：程序内固定的示例时间
         * 操作：
         * 1) 创建时间对象
         * 2) 转换或计算并检查结果
         */
        logger.info("开始处理时间");
        // 1.1 创建固定 UTC 时刻
        Instant instant = Instant.parse("2024-01-01T00:00:00Z");
        // 1.2 读取秒、毫秒并验证往返
        long seconds = instant.getEpochSecond();
        long millis = instant.toEpochMilli();
        System.out.println("seconds=" + seconds);
        System.out.println("millis=" + millis);
        System.out.println("same=" + Instant.ofEpochMilli(millis).equals(instant));
        logger.info("时间处理完成");
    }
}
```

标准输出：

```text
seconds=1704067200
millis=1704067200000
same=true
```

## 执行过程

固定时刻是 2024-01-01 00:00 UTC。`getEpochSecond()` 给出从 1970-01-01T00:00:00Z 起的秒数，`toEpochMilli()` 给出毫秒数；数值相差 1000 倍。该例没有亚毫秒部分，所以从毫秒还原后与原值相等。

如果 Instant 包含更细的纳秒部分，转换成毫秒可能丢失精度，往返不一定相等。超出目标毫秒 long 的可表示范围时转换也可能失败；接口和数据库必须明确单位、精度和范围。

## 工作中怎么用

`Instant.now()` 取当前时刻，生产系统可能受系统时钟调整；测量耗时通常应使用适合测量间隔的单调时钟 API，如 `System.nanoTime()`，不能把它当 UTC 时间戳。为业务测试注入 `Clock`，避免依赖真实当前时间。

向用户展示时，使用明确的 `ZoneId` 转成 ZonedDateTime；不要把 Instant 直接当作某地区本地时间。日志与数据库传输应约定是否为 UTC、单位及精度。

## 手练与答案

`Instant.ofEpochSecond(1)` 表示 epoch 后一秒；`Instant.ofEpochMilli(1)` 表示 epoch 后一毫秒，两者不是同一时刻。本例毫秒值是 1704067200000，不应传给接收秒的 API。

参考：[廖雪峰：Instant](https://liaoxuefeng.com/books/java/datetime/instant/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/Instant.html)。
