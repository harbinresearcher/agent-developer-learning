# 使用Commons Logging

> **重要程度：低**｜**手练：按需**
> **重点**：日志门面与具体实现的分工。

## 核心概念

Commons Logging 是日志门面，应用或库通过门面记录日志，实际输出由合适的底层日志实现和配置完成。门面与实现是不同角色，缺少或冲突的适配器会影响最终输出。

## 工作中注意

新旧项目使用的日志技术可能不同；维护时先查看依赖树和实际绑定，不要在同一进程里随意添加多个实现或互相桥接形成循环。

## 手练

在已有项目中定位 Commons Logging 的依赖来源和实际日志实现；不要为了练习改动生产配置。

来源：[教程：使用Commons Logging](https://liaoxuefeng.com/books/java/exception/commons-logging/index.html)。
