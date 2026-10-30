# Spring：IoC、AOP、事务与 MVC

> **重要程度：高**｜**手练：需要**
> **重点**：依赖注入、Bean 生命周期、代理边界、事务生效条件、请求处理链路。

## IoC 与 Bean

Spring 容器负责创建和管理 Bean，并按配置注入依赖。构造器注入让必需依赖在创建时明确；自动装配、条件装配、配置属性和不同 scope 都会影响最终对象。排错时先看容器实际创建了哪些 Bean，以及当前激活的配置。

## AOP 与事务

AOP 常通过代理在方法调用前后加入事务、日志等行为。`@Transactional` 的效果取决于事务管理器、代理调用路径、传播行为、异常规则等；同类内部直接调用通常绕过基于代理的拦截，不能只看到注解就断言事务生效。默认回滚规则与项目配置也要核对。

数据访问可使用 JDBC、MyBatis、JPA 等方案。ORM 不免除对 SQL、事务、索引和查询数量的理解；留意 N+1 查询和连接耗尽。

## MVC 与 API

请求经 DispatcherServlet 路由到控制器，完成参数绑定、校验、业务调用和响应转换。控制器不应承载所有业务和数据访问逻辑。REST API 应明确资源、状态码、错误格式与权限；跨域 CORS 不是身份认证，Filter 和 Interceptor 的拦截位置与能力也不同。

其他集成如邮件、消息和定时任务要明确重试、幂等、超时、失败告警和关闭行为。不要在请求中无界等待外部服务。

## 手练

编写一个包含控制器、服务和数据访问层的小接口；为一次多步骤更新加入事务，分别从代理外部和同类内部调用，验证回滚行为。

来源：[教程：Spring开发](https://liaoxuefeng.com/books/java/spring/index.html)；[Spring 事务代理](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)。
