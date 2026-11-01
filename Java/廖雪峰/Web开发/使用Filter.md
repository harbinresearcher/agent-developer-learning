# 使用Filter

> **重要程度：高**｜**手练：建议**
> **重点**：请求链和过滤顺序。

## 核心概念

Filter 在请求进入目标资源之前或之后处理通用逻辑，可组成过滤链。

## 工作中注意

顺序会影响鉴权、编码和日志；调用链及异常路径都要验证，避免重复读取请求体。

## 手练

添加一个请求耗时 Filter，确认正常和异常路径都会记录结果。

来源：[教程：使用Filter](https://liaoxuefeng.com/books/java/web/filter/index.html)。

## 关联知识与实践

Filter 可以在请求到达目标前后处理通用逻辑；Listener 监听容器或会话等生命周期事件。JSP 在旧项目中可能遇到，新项目是否采用模板渲染、前后端分离或其他方案取决于项目。部署时确认线程、连接、配置、日志和健康检查。

来源：[教程：Web开发](https://liaoxuefeng.com/books/java/web/index.html)；[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
