# Lambda基础

> **重要程度：高**｜**手练：需要**
> **重点**：函数式接口、参数与返回值、捕获变量。

## 解决什么问题

有时需要把一段可执行规则作为参数传给别的方法，例如筛选通过的成绩。Lambda 是实现函数式接口的一种简洁写法；`Predicate<Integer>` 表示接收 Integer 并返回 boolean 的判断。

## 完整例子

保存为 `LambdaDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.logging.Logger;
public class LambdaDemo {
    private static final Logger logger = Logger.getLogger(LambdaDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：使用 Lambda 判断分数
         * ========================================================================
         * 数据源：程序中准备的固定示例数据
         * 操作：
         * 1) 创建函数或数据流
         * 2) 处理输入并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 定义通过阈值和判断函数
        int passing = 60;
        Predicate<Integer> isPassing = score -> score >= passing;
        // 1.2 对两个输入执行判断
        System.out.println("59=" + isPassing.test(59));
        System.out.println("60=" + isPassing.test(60));
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
59=false
60=true
```

## 执行过程

`score -> score >= passing` 声明一个输入和判断表达式。目标类型 `Predicate<Integer>` 决定输入和返回类型；调用 `test(59)` 得到 false，`test(60)` 得到 true。Lambda 捕获外层局部变量 passing，它在创建后没有重新赋值，属于 effectively final。

函数式接口有一个抽象方法；它仍可包含符合规则的 default、static 方法等。常见接口包括 `Function<T,R>` 转换值、`Predicate<T>` 判断、`Consumer<T>` 接收并处理、`Supplier<T>` 提供值。复杂业务不一定适合写成长 Lambda，提取具名方法可改善可读性。

## 工作中注意

捕获的局部变量必须为 final 或事实上的 final；这不等于捕获引用指向的对象一定不可变。Lambda 可以产生副作用，但用于 Stream 等操作时依赖共享可变状态可能造成并发或执行顺序问题。注意自动拆箱：此例若传 null，比较时可能抛空指针异常。

## 手练与答案

把阈值改为 70：59 和 60 均为 false。若在 Lambda 捕获 passing 后重新给该局部变量赋值，编译器会拒绝，因为它不再是 effectively final。

参考：[廖雪峰：Lambda基础](https://liaoxuefeng.com/books/java/functional/basic/index.html)；[Oracle Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
