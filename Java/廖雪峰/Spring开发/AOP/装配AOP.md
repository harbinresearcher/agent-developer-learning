# 装配AOP

> **重要程度：高**｜**手练：需要**
> **重点**：切点匹配、前置通知、正常返回与异常通知。

## 切点和通知分别负责什么

切点决定哪些方法匹配，通知决定匹配后何时做什么。例子的 `execution(* learn.advice.ReportService.create(..))` 选择该类的 `create` 方法：第一个 `*` 表示返回类型不限，`(..)` 表示参数列表不限。表达式应尽量限定到明确的业务边界，避免误拦截无关方法。

`@Before` 在目标调用前执行；`@AfterReturning` 在目标正常返回后执行；`@AfterThrowing` 在目标抛出匹配异常后执行。示例使用 Spring 自动代理，切面和目标都通过 `@Bean` 注册。

## 完整例子

使用 JDK 17、Maven、Spring Framework 7.0.10 与 AspectJ 注解支持。保存 `pom.xml`：

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

代码保存为 `src/main/java/learn/advice/AdviceOrderDemo.java`：

```java
package learn.advice;
import java.util.logging.Logger;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
class ReportService {
    public String create(boolean fail) {
        System.out.println("执行业务");
        if (fail) throw new IllegalStateException("模拟失败");
        return "完成";
    }
}
@Aspect
class ReportAdvice {
    private static final Logger logger = Logger.getLogger(ReportAdvice.class.getName());
    /* ========================================================================
     * 步骤1：在目标调用前后观察结果
     * 数据源：切点匹配到的 ReportService.create 方法
     * 操作：1) 记录调用前 2) 区分正常返回与异常
     * ======================================================================== */
    @Before("execution(* learn.advice.ReportService.create(..))")
    public void before() {
        logger.info("开始前置通知");
        // 1.1 目标调用前执行
        System.out.println("前置通知");
        logger.info("前置通知结束");
    }
    @AfterReturning("execution(* learn.advice.ReportService.create(..))")
    public void success() {
        logger.info("开始成功通知");
        // 1.2 目标正常返回后执行
        System.out.println("成功通知");
        logger.info("成功通知结束");
    }
    @AfterThrowing("execution(* learn.advice.ReportService.create(..))")
    public void failure() {
        logger.info("开始异常通知");
        // 1.3 目标抛出异常后执行
        System.out.println("异常通知");
        logger.info("异常通知结束");
    }
}
public class AdviceOrderDemo {
    private static final Logger logger = Logger.getLogger(AdviceOrderDemo.class.getName());
    @Configuration @EnableAspectJAutoProxy
    static class Config {
        @Bean ReportService reportService() { return new ReportService(); }
        @Bean ReportAdvice reportAdvice() { return new ReportAdvice(); }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：比较两种调用结果
         * 数据源：容器中的报告服务代理
         * 操作：1) 正常调用 2) 模拟异常并在调用方捕获
         * ======================================================================== */
        logger.info("开始比较通知");
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class)) {
            // 1.1 正常完成时观察成功通知
            ReportService service = context.getBean(ReportService.class);
            service.create(false);
            // 1.2 失败时观察异常通知和异常传播
            try { service.create(true); }
            catch (IllegalStateException ex) { System.out.println("调用方捕获: " + ex.getMessage()); }
        }
        logger.info("通知比较结束");
    }
}
```

先用 `java -version` 和 `mvn -v` 确认都使用 JDK 17 或以上。PowerShell 在项目根目录运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$springDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
java -cp "target/classes;$springDeps" learn.advice.AdviceOrderDemo
```

标准输出如下，日志单独显示：

```text
前置通知
执行业务
成功通知
前置通知
执行业务
异常通知
调用方捕获: 模拟失败
```

## 根据输出理解边界

第一次目标正常返回，所以运行前置和成功通知；第二次目标抛异常，所以运行前置和异常通知，最后异常仍由调用方捕获。异常通知不会自动把异常变成成功结果。`@After` 类似 finally，适合无论成功与否都需要执行的清理；`@Around` 能控制是否和何时调用目标，能力更大，也更容易意外改变业务。

多个切面共同匹配时，顺序需要明确配置并用测试验证；不要根据类名或声明顺序猜测。前置通知本身抛异常也可能阻止目标执行。日志和审计通知不应泄漏敏感参数。

## 动手与自查

1. 改错切点方法名，验证目标仍执行，但三个通知都不再匹配。
2. 增加 `@After`，分别检查正常和异常两次调用；若添加其他通知，要验证实际顺序。
3. 将通知改为 `@Around`，解释忘记 `proceed()` 会带来什么后果。

来源：[教程：装配AOP](https://liaoxuefeng.com/books/java/spring/aop/basic/index.html)、[Spring AOP 通知](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/advice.html)。
