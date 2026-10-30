# LocalDateTime

> **重要程度：高**｜**手练：需要**
> **重点**：本地日期时间、不含时区。

## 核心概念

LocalDateTime 保存日期和钟表时间，不包含时区或偏移；适合表达尚未绑定地区的本地时间。

## 工作中注意

需要变成时间点时必须结合地区时区，遇到夏令时跳变或重复时刻还需明确处理规则。

## 手练

解析本地日期时间，为其指定地区时区，再转换成 Instant。

来源：[教程：LocalDateTime](https://liaoxuefeng.com/books/java/datetime/local-datetime/index.html)。
