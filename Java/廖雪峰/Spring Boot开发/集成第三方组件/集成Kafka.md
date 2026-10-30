# 集成Kafka

> **重要程度：中**｜**手练：按需**
> **重点**：主题、分区和消费进度。

## 核心概念

Kafka 以主题和分区组织事件，消费者按消费组处理记录并维护消费进度。

## 工作中注意

顺序保证有明确范围；重复处理、偏移提交、重试和业务幂等需要联合设计。

## 手练

向测试主题写入几条事件，记录分区和消费进度并模拟重启。

来源：[教程：集成Kafka](https://liaoxuefeng.com/books/java/springboot/integration/kafka/index.html)。
