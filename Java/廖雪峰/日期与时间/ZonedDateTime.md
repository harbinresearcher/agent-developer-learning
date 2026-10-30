# ZonedDateTime

> **重要程度：高**｜**手练：需要**
> **重点**：地区时区和时间转换。

## 核心概念

ZonedDateTime 将日期时间与地区时区规则结合，可通过 ZoneId 表达 Asia/Shanghai 等地区。

## 工作中注意

地区时区和固定偏移不完全相同；转换时要确认是保持同一时间点，还是保持同一本地钟表时间。

## 手练

把同一时刻从上海转换到纽约，确认 Instant 未变化而展示时间变化。

来源：[教程：ZonedDateTime](https://liaoxuefeng.com/books/java/datetime/zoned-datetime/index.html)。
