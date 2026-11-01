# 使用AOP

> **重要程度：高**｜**手练：需要**
> **重点**：代理、切点、通知、目标方法调用和适用边界。

## AOP 解决什么问题

多个业务方法可能都需要记录耗时、权限检查或事务边界。如果每个方法都重复这些处理，业务代码会被横向职责占据。AOP 可以把共同处理集中，再按规则应用到符合条件的方法。

Spring AOP 通常通过代理工作：调用者先调用代理；匹配的通知包围目标方法；通知决定何时调用目标并如何处理返回值或异常。它不会自动让所有对象、所有方法都获得这些行为。

## 完整例子

本例用 `ProxyFactory` 显式创建代理，观察底层原理。生产应用通常让 Spring 容器按配置创建代理。使用 JDK 17、Maven 和 Spring Framework 7.0.10，保存以下 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>learn.example</groupId><artifactId>spring-ioc-demo</artifactId><version>1.0</version>
    <properties><maven.compiler.release>17</maven.compiler.release><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding></properties>
    <dependencies>
        <dependency><groupId>org.springframework</groupId><artifactId>spring-context</artifactId><version>7.0.10</version></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

将代码保存为 `src/main/java/AopIntroDemo.java`：

```java
import java.util.logging.Logger;
import org.aopalliance.intercept.MethodInterceptor;
import org.springframework.aop.framework.ProxyFactory;
public class AopIntroDemo {
    private static final Logger logger = Logger.getLogger(AopIntroDemo.class.getName());
    interface PriceService { int total(int price, int count); }
    static class PriceServiceImpl implements PriceService {
        public int total(int price, int count) { return price * count; }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：包装计价服务
         * 数据源：计价目标对象与拦截器
         * 操作：1) 创建代理 2) 调用业务方法并返回结果
         * ======================================================================== */
        logger.info("开始创建代理");
        // 1.1 包装目标对象，在方法调用前后增加日志
        ProxyFactory factory = new ProxyFactory(new PriceServiceImpl());
        factory.setInterfaces(PriceService.class);
        factory.addAdvice((MethodInterceptor) call -> {
            logger.info("开始调用计价方法");
            try {
                // 1.2 调用目标并原样返回结果
                return call.proceed();
            } finally {
                logger.info("计价方法调用结束");
            }
        });
        PriceService service = (PriceService) factory.getProxy();
        System.out.println("金额: " + service.total(25, 4));
        logger.info("代理调用完成");
    }
}
```

先运行 `java -version` 与 `mvn -v`，确认都使用 JDK 17 或以上。PowerShell 在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" AopIntroDemo
```

标准输出如下；拦截器的开始和结束日志显示在日志输出中：

```text
金额: 100
```

## 按调用顺序理解

`service.total(25, 4)` 先到代理，代理运行拦截器；`call.proceed()` 再进入目标对象计算 25 × 4，结果返回拦截器，最后返回调用方。`finally` 在成功和抛异常时都记录结束日志。本例为便于说明金额使用整数；真实金额需要单位、精度与舍入规则。

切面（Aspect）是横切逻辑的组织单元；通知（Advice）是具体动作；切点（Pointcut）筛选拦截位置；连接点是可应用通知的位置。Spring AOP 以方法执行为主要连接点；完整 AspectJ 支持的连接点更广。

## 工作中怎么用

- AOP 适合与多个业务方法有关、语义一致的横切逻辑；具体订单计算仍应留在业务服务中。
- 事务、鉴权和审计优先使用项目的成熟框架能力，并用测试确认顺序、异常及边界。切面不应打印密码、凭据或完整敏感参数。
- 容器代理的同类自调用可能绕过通知；直接 `new` 的对象通常不自动受代理管理。代理类型和可拦截方法也受配置及类结构影响。

## 动手与自查

1. 在目标方法抛出异常，检查结束日志是否出现以及异常是否传播。
2. 暂时移除 `call.proceed()`，解释目标方法是否执行、返回值如何变化。
3. 为什么直接调用 `new PriceServiceImpl().total(...)` 不会自动经过例子里的拦截器？

来源：[教程：使用AOP](https://liaoxuefeng.com/books/java/spring/aop/index.html)、[Spring AOP 概念](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html)。
