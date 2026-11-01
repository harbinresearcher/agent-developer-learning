# for 循环

> **重要程度：高**｜**手练：需要**
> **重点**：下标边界、执行顺序、增强 for。

## 先理解：把循环控制放在一处

`for (初始化; 条件; 更新) { 循环体 }` 先执行一次初始化；每轮先检查条件，为真则运行循环体，然后执行更新，再回到条件处。条件首次为假时，循环体一次也不运行。

## 完整例子：统计成绩

保存为 `ForDemo.java`，编译并运行；日志写到标准错误。

```java
import java.util.logging.Logger;

public class ForDemo {
    private static final Logger logger = Logger.getLogger(ForDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：遍历数组并统计
         * ========================================================================
         * 数据源：程序内准备的示例数据
         * 操作：
         * 1) 初始化输入
         * 2) 按控制流程处理并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备数组和累计值
        int[] scores = {80, 90, 70};
        int sum = 0;
        // 1.2 用下标依次访问每个元素
        for (int i = 0; i < scores.length; i++) {
            sum += scores[i];
        }
        // 1.3 输出累计值和元素个数
        System.out.println("sum=" + sum + ", count=" + scores.length);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
sum=240, count=3
```

## 按顺序读代码

`scores.length` 是 3，合法下标是 0、1、2。`i=0` 时加 80，更新为 1；随后加 90、70。`i` 更新为 3 后，`3 < 3` 为假，结束。`i` 在 for 头部声明，只在这个 for 及其循环体的作用域内可用。

边界应写 `i < scores.length`。如果误写 `i <= scores.length`，循环会访问 `scores[3]`，抛出 `ArrayIndexOutOfBoundsException`。空数组长度为 0，因此这里循环 0 次，和为 0；算平均分时必须先判断长度，避免整数除零。

## 增强 for 与选择

只读取每个元素时，可以用增强 for：`for (int score : scores)`，依次把每个元素值赋给 `score`。它不直接提供下标。对 `int[]` 修改循环变量 `score` 不会修改数组元素；需要原地修改时用下标 `scores[i] = ...`。增强 for 还可遍历实现 `Iterable` 的对象，相关细节见 [[使用Iterator]]。

## 手练与答案

1. 改成 `i = 1` 开始：只加 90 和 70，结果为 160。
2. 用空数组：结果为 `sum=0, count=0`。若求平均分，先处理没有成绩的情况。

课程原文：[廖雪峰 Java 教程：for 循环](https://liaoxuefeng.com/books/java/quick-start/flow/for/index.html)。
