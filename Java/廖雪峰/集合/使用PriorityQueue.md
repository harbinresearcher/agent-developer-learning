# 使用PriorityQueue

> **重要程度：中**｜**手练：建议**
> **重点**：优先级、比较器和遍历顺序。

## 核心概念

PriorityQueue 按自然顺序或比较器决定队头，`poll` 返回当前优先级最高的元素；默认自然顺序下通常是最小元素先出。

## 工作中注意

遍历优先队列不保证展示完整排序；若需要排序输出，应依次取出或复制后排序。放入后不要随意改变影响优先级的字段。

## 手练

按截止时间处理三项任务，检查逐次 poll 的顺序，再比较直接遍历结果。

来源：[教程：使用PriorityQueue](https://liaoxuefeng.com/books/java/collection/priority-queue/index.html)。
