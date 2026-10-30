# 集成RabbitMQ

> **重要程度：中**｜**手练：按需**
> **重点**：交换机、队列和确认。

## 核心概念

RabbitMQ 使用交换机和队列等机制路由消息，应用通过客户端发布和消费。

## 工作中注意

确认、重试、死信和幂等由具体配置与业务共同决定；不要声称消息天然只消费一次。

## 手练

发送测试消息并模拟消费者失败，观察确认和重试。

来源：[教程：集成RabbitMQ](https://liaoxuefeng.com/books/java/springboot/integration/rabbitmq/index.html)。
