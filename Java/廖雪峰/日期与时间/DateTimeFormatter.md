# DateTimeFormatter

> **重要程度：高**｜**手练：需要**
> **重点**：格式化、解析与年份格式。

## 核心概念

DateTimeFormatter 按指定格式和地区解析或格式化日期时间；格式符号区分大小写。

## 工作中注意

日历年与周基年不是一回事，跨年时 `YYYY` 和 `yyyy` 可能不同；严格解析还要考虑 `uuuu` 与解析规则。

## 手练

格式化跨年附近的日期，比较周基年与日历年结果。

来源：[教程：DateTimeFormatter](https://liaoxuefeng.com/books/java/datetime/datetime-formatter/index.html)。
