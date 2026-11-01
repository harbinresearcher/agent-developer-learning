# 使用注解装配AOP

> **重要程度：高**｜**手练：需要**
> **重点**：切点匹配、环绕通知、代理调用、返回值与异常传播。

## 先理解几个词

AOP 把日志、事务、权限等横跨多个业务方法的处理集中管理。目标对象提供业务方法；切面（Aspect）封装横切逻辑；切点（Pointcut）选择要拦截的方法；通知（Advice）是在匹配处运行的代码。Spring AOP 通常依靠代理拦截方法调用。

例子通过 `@Aspect` 声明切面，`@Around` 指定切点，`@EnableAspectJAutoProxy` 启用自动代理；切面本身也必须成为容器 Bean。Spring 使用 AspectJ 注解和表达式不等于示例使用了 AspectJ 字节码织入。

## 完整例子

使用 JDK 17、Maven 和 Spring Framework 7.0.10。保存以下 `pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>learn.example</groupId><artifactId>spring-ioc-demo</artifactId><version>1.0</version>
    <properties><maven.compiler.release>17</maven.compiler.release><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding></properties>
    <dependencies>
        <dependency><groupId>org.springframework</groupId><artifactId>spring-context</artifactId><version>7.0.10</version></dependency>
        <dependency><groupId>org.aspectj</groupId><artifactId>aspectjweaver</artifactId><version>1.9.25.1</version></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

保存代码为 `src/main/java/learn/aop/AopAnnotationDemo.java`：

```java
package learn.aop;
import java.util.logging.Logger;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
class OrderService {
    public String place() { System.out.println("处理订单"); return "成功"; }
}
@Aspect
class TimingAspect {
    private static final Logger logger = Logger.getLogger(TimingAspect.class.getName());
    @Around("execution(* learn.aop.OrderService.place(..))")
    public Object around(ProceedingJoinPoint call) throws Throwable {
        /* ========================================================================
         * 步骤1：包围目标方法
         * 数据源：拦截到的订单方法调用
         * 操作：1) 调用目标 2) 在 finally 记录耗时
         * ======================================================================== */
        logger.info("开始调用订单方法");
        long start = System.nanoTime();
        try {
            // 1.1 放行目标调用并保留其返回值或异常
            System.out.println("调用前");
            Object result = call.proceed();
            System.out.println("正常返回后");
            return result;
        } finally {
            // 1.2 无论成功或失败都记录耗时
            logger.info("订单方法调用结束，耗时纳秒: " + (System.nanoTime() - start));
        }
    }
}
public class AopAnnotationDemo {
    private static final Logger logger = Logger.getLogger(AopAnnotationDemo.class.getName());
    @Configuration
    @EnableAspectJAutoProxy
    static class Config {
        @Bean OrderService orderService() { return new OrderService(); }
        @Bean TimingAspect timingAspect() { return new TimingAspect(); }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：调用容器中的订单服务
         * 数据源：Spring 管理的服务与切面
         * 操作：1) 创建容器 2) 从代理调用目标
         * ======================================================================== */
        logger.info("开始创建容器");
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            // 1.1 获取容器处理后的服务
            OrderService service = context.getBean(OrderService.class);
            // 1.2 调用目标并输出原返回值
            System.out.println("结果: " + service.place());
        }
        logger.info("容器调用结束");
    }
}
```

PowerShell 在项目根目录运行：

先运行 `java -version` 和 `mvn -v`，确认两者都使用 JDK 17 或以上；不一致时先调整 `JAVA_HOME` 和 `PATH`。

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" learn.aop.AopAnnotationDemo
```

标准输出，日志另行显示：

```text
调用前
处理订单
正常返回后
结果: 成功
```

## 调用链和异常

容器发现 `OrderService.place` 匹配切点，返回可拦截的代理。调用者进入 `around`，`proceed()` 调用目标，目标返回后通知再返回原结果。目标抛异常时，本例的“正常返回后”不会执行，但 `finally` 仍记录耗时；没有捕获异常，所以异常继续传播。

环绕通知不调用 `proceed()` 会阻止目标方法执行；多次调用会重复执行业务，重试前要考虑幂等。`System.nanoTime()` 适合测量经过时间，不代表墙上时间；示例耗时包括目标方法及相关调用。

## 工作中排错

- 切面没有生效时，检查切面是否为 Bean、自动代理是否启用、表达式是否匹配，以及调用是否经过代理。
- 同一对象内部的 `this` 自调用通常绕过基于代理的拦截；`new` 出的普通对象也不会自动获得切面。
- JDK 接口代理与类代理有不同限制；`final` 类或方法不能按常规类代理方式被覆盖。生产代码优先用框架提供的事务、安全和指标能力，日志不要泄露参数中的敏感数据。

## 动手与自查

1. 让 `place()` 抛出异常，观察后置输出、finally 日志和异常传播。
2. 把切点的方法名改错，确认业务仍能调用但通知不执行。
3. 删掉 `proceed()`，预测再验证目标方法是否执行；说明为什么这会改变业务行为。

来源：[教程：使用注解装配AOP](https://liaoxuefeng.com/books/java/spring/aop/annotation-config/index.html)、[Spring 启用 AspectJ 注解支持](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/aspectj-support.html)、[Spring 代理机制](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)。
