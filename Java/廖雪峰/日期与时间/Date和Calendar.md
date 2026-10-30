# Date和Calendar

> **重要程度：中**｜**手练：按需**
> **重点**：旧 API、时间戳和兼容。

## 核心概念

Date 和 Calendar 是旧日期时间 API，维护老项目时仍可能遇到。Date 表示时间点，格式化和展示需要额外时区处理。

## 工作中注意

不要只看 Date 默认字符串输出来判断时区；新代码通常优先使用 java.time，旧接口互转时确认精度和时区。

## 手练

将一个旧 Date 转为 Instant，并在指定时区显示，检查时间点是否改变。

来源：[教程：Date和Calendar](https://liaoxuefeng.com/books/java/datetime/date-calendar/index.html)。
