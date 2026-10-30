# 使用Interceptor

> **重要程度：中**｜**手练：建议**
> **重点**：MVC 拦截位置与生命周期。

## 核心概念

Interceptor 在 Spring MVC 处理器调用前后提供拦截点，适合与 MVC 处理过程相关的逻辑。

## 工作中注意

它与 Servlet Filter 所处层次和覆盖范围不同；安全关键检查应按框架官方建议配置。

## 手练

为测试控制器记录拦截顺序，比较 Filter 和 Interceptor 的位置。

来源：[教程：使用Interceptor](https://liaoxuefeng.com/books/java/spring/web/interceptor/index.html)。
