# Instant

> **重要程度：高**｜**手练：需要**
> **重点**：时间线、UTC 与时间戳精度。

## 核心概念

Instant 表示时间线上的一个时刻，适合记录事件发生时间和在系统间传递时间点。

## 工作中注意

秒、毫秒和纳秒不能混用；输出面向用户的本地时间时，需结合目标时区。

## 手练

将当前 Instant 转为 epoch 毫秒并还原，再转换为指定时区。

来源：[教程：Instant](https://liaoxuefeng.com/books/java/datetime/instant/index.html)。
