# do while 循环

> **重要程度：高**｜**手练：需要**
> **重点**：至少执行一次和退出时机。

## 先理解：至少执行一次

`do { 循环体 } while (条件);` 先运行循环体，再判断条件。即使条件一开始就为假，循环体仍会执行一次。注意末尾有分号；条件也必须是 `boolean`。

## 完整例子

保存为 `DoWhileDemo.java`，编译并运行；日志写到标准错误。

```java
import java.util.logging.Logger;

public class DoWhileDemo {
    private static final Logger logger = Logger.getLogger(DoWhileDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：先处理再判断
         * ========================================================================
         * 数据源：程序内准备的示例数据
         * 操作：
         * 1) 初始化输入
         * 2) 按控制流程处理并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备计数器
        int n = 4;
        int count = 0;
        // 1.2 至少处理一次，再检查是否继续
        do {
            count++;
            n++;
        } while (n <= 3);
        // 1.3 输出处理次数和最终值
        System.out.println("count=" + count + ", n=" + n);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
count=1, n=5
```

## 按顺序读代码

初始 `n=4`、`count=0`。先进入 `do`，`count` 增为 1，`n` 增为 5；之后判断 `5 <= 3` 为假，退出。因此输出 `count=1, n=5`。如果改为 `while (n <= 3)` 包住同一段循环体，第一次检查就为假，输出会是 `count=0, n=4`。

## 工作中怎么用

当业务必须先做一次再决定是否重复时，可以用它，例如先展示一次菜单，再根据用户输入决定是否重新显示。若第一次就可能不应处理，使用 `while` 更符合意图。无论哪种循环，都要确认条件会变化并能退出。

## 手练与答案

将初始 `n` 改成 2：第一轮后 `n=3`，条件为真；第二轮后 `n=4`，条件为假。最终输出 `count=2, n=4`。

课程原文：[廖雪峰 Java 教程：do while 循环](https://liaoxuefeng.com/books/java/quick-start/flow/do-while/index.html)。
