# 使用Spring MVC

> **重要程度：高**｜**手练：需要**
> **重点**：DispatcherServlet、路由和绑定。

## 核心概念

Spring MVC 中 DispatcherServlet 协调请求映射、参数绑定、处理器调用和响应。

## 工作中注意

控制器应负责协议边界，不应承担全部数据库与业务逻辑；检查请求映射冲突。

## 手练

设计 Controller、Service 和 DAO 的调用链，并为接口编写测试。

来源：[教程：使用Spring MVC](https://liaoxuefeng.com/books/java/spring/web/mvc/index.html)。

## 关联知识与实践

请求经 DispatcherServlet 路由到控制器，完成参数绑定、校验、业务调用和响应转换。控制器不应承载所有业务和数据访问逻辑。REST API 应明确资源、状态码、错误格式与权限；跨域 CORS 不是身份认证，Filter 和 Interceptor 的拦截位置与能力也不同。

来源：[教程：Spring开发](https://liaoxuefeng.com/books/java/spring/index.html)；[Spring 事务代理](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)。
