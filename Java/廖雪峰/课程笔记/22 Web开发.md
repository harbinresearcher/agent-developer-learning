# Web 开发：Servlet 与请求链路

> **重要程度：高**｜**手练：需要**
> **重点**：请求响应、Servlet 生命周期、Cookie 与 Session、Filter、部署和安全。

## 请求如何到达业务代码

浏览器或客户端发送 HTTP 请求，Web 服务器接收请求并交给 Servlet 容器；容器按照映射找到 Servlet 或 Filter，再调用应用代码。响应包含状态码、头部和正文。Servlet 对象通常由容器管理，请求可并发到达；不要把每个请求的数据存放在共享的可变实例字段中。

重定向由客户端再次发起请求，地址栏通常变化；转发在服务端内部交给其他组件处理，客户端通常不知道内部转发。Cookie 保存在客户端；Session 是服务端的一种会话状态，通常借助会话标识关联请求。Cookie 的 `HttpOnly`、`Secure`、`SameSite` 应按部署和安全需求配置；不要把敏感数据直接放到未保护的 Cookie。

Filter 可以在请求到达目标前后处理通用逻辑；Listener 监听容器或会话等生命周期事件。JSP 在旧项目中可能遇到，新项目是否采用模板渲染、前后端分离或其他方案取决于项目。部署时确认线程、连接、配置、日志和健康检查。

Java EE 已演进为 Jakarta EE。现代版本常使用 `jakarta.*` 命名空间；旧教程或项目可能仍使用 `javax.*`，不能不核对框架版本就混用。

## 手练

实现一个简单请求处理器，读取查询参数并返回结果；比较重定向和转发，再用 Filter 记录请求耗时，注意不记录敏感参数。

来源：[教程：Web开发](https://liaoxuefeng.com/books/java/web/index.html)；[Jakarta Servlet 规范](https://jakarta.ee/specifications/servlet/)。
