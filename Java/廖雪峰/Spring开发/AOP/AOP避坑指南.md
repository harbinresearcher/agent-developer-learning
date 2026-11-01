# AOP避坑指南

> **重要程度：高**｜**手练：需要**
> **重点**：调用是否经过代理、自调用、代理限制、异常和事务边界。

## 为什么注解写了却没有生效

Spring AOP 默认通过代理拦截调用。只有调用经过适当的代理且切点匹配，通知才有机会运行。目标对象内部的 `this.inner()` 直接调用目标自身，不会再次进入代理。这会影响使用代理实现的事务、缓存或异步等能力，但具体行为仍要结合所用注解和配置判断。

下面用 `ProxyFactory` 手工创建代理，专门观察调用路径。它展示的是代理原理；真实项目通常由容器创建代理。

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
    </dependencies>
    <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

代码保存为 `src/main/java/AopSelfCallDemo.java`：

```java
import java.util.logging.Logger;
import org.aopalliance.intercept.MethodInterceptor;
import org.springframework.aop.framework.ProxyFactory;
public class AopSelfCallDemo {
    private static final Logger logger = Logger.getLogger(AopSelfCallDemo.class.getName());
    interface Work { void outer(); void inner(); }
    static class WorkImpl implements Work {
        public void outer() {
            System.out.println("目标: outer");
            this.inner();
        }
        public void inner() { System.out.println("目标: inner"); }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：创建代理并对比调用路径
         * 数据源：实现 Work 的目标对象
         * 操作：1) 添加方法拦截器 2) 比较内部调用与外部调用
         * ======================================================================== */
        logger.info("开始代理实验");
        // 1.1 创建接口代理，为每次经过代理的方法打印通知
        ProxyFactory factory = new ProxyFactory(new WorkImpl());
        factory.setInterfaces(Work.class);
        factory.addAdvice((MethodInterceptor) call -> {
            logger.info("开始拦截方法");
            try {
                System.out.println("通知: " + call.getMethod().getName());
                return call.proceed();
            } finally {
                logger.info("拦截方法结束");
            }
        });
        Work proxy = (Work) factory.getProxy();
        // 1.2 对比目标内部调用与直接从代理调用
        proxy.outer();
        proxy.inner();
        logger.info("代理实验结束");
    }
}
```

先运行 `java -version` 和 `mvn -v`，确认两者都使用 JDK 17 或以上；不一致时先调整 `JAVA_HOME` 和 `PATH`。PowerShell 在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" AopSelfCallDemo
```

标准输出（日志单独显示）：

```text
通知: outer
目标: outer
目标: inner
通知: inner
目标: inner
```

## 按顺序看调用

`proxy.outer()` 先进入代理，因此打印 `通知: outer`。目标 `outer()` 内部的 `this.inner()` 直接调用目标，打印 `目标: inner`，中间没有 `通知: inner`。主方法随后调用 `proxy.inner()`，这一次经过代理，于是先打印通知。

优先将需要独立拦截的行为拆到另一个由容器管理的对象，再从外部通过其代理调用；先确定业务职责和事务边界。不要依赖把 `this` 强转成代理。AspectJ 字节码织入采用不同机制，不应把本例代理结论套到所有 AOP 实现。

## 其他常见问题

- 手动 `new` 的对象通常不经过容器自动代理；切面本身、配置和目标都要被正确管理。
- JDK 动态代理主要代理接口；类代理依赖继承，`final` 类和不能覆盖的方法有相应限制。Spring Boot 和框架的代理默认配置可能不同。
- 切点没有匹配、方法不可拦截、通知顺序或异常被吞掉，都可能改变实际效果。环绕通知应按业务决定是否调用 `proceed()` 并正确传播结果。
- 事务方法内把异常吃掉可能影响回滚判断；跨线程或跨进程调用也不能想当然地继承当前事务上下文。用集成测试观察实际数据库结果。

## 动手与自查

1. 自己画出 `proxy.outer → target.outer → this.inner` 的链路，并对照五行输出。
2. 把 `inner` 移到另一个代理对象中，从 `outer` 调用它，验证通知出现。
3. 在真实 Spring 事务项目中比较自调用和代理外部调用，用数据库结果而不是注解推断是否回滚。

来源：[教程：AOP避坑指南](https://liaoxuefeng.com/books/java/spring/aop/notice/index.html)、[Spring 代理机制](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)。
