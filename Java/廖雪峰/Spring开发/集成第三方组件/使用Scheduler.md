# 使用Scheduler

> **重要程度：中**｜**手练：建议**
> **重点**：调度、执行与重复。

## 核心概念

调度器按配置触发任务；任务实际执行与安排下一次执行是不同阶段。

## 工作中注意

多实例部署、任务超时和进程重启可能造成重复或遗漏；业务需明确幂等与并发策略。

## 手练

安排一个测试任务并观察触发时间，模拟执行重叠并记录处理方式。

来源：[教程：使用Scheduler](https://liaoxuefeng.com/books/java/spring/integration/scheduler/index.html)。
