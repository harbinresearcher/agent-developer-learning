# 定制Bean

> **重要程度：中**｜**手练：建议**
> **重点**：生命周期、初始化与销毁。

## 核心概念

Bean 可以按框架支持的方式定制初始化、销毁及作用域；容器负责相应生命周期管理。

## 工作中注意

初始化失败需要清理已分配资源；prototype 等作用域的销毁责任可能不同。

## 手练

为测试资源配置初始化和清理，关闭容器后检查资源状态。

来源：[教程：定制Bean](https://liaoxuefeng.com/books/java/spring/ioc/customize/index.html)。
