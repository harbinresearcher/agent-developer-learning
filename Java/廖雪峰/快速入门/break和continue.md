# break和continue

> **重要程度：高**｜**手练：需要**
> **重点**：跳过本轮、退出循环与嵌套边界。

## 先理解：跳过和结束不是一回事

`break` 结束当前所在的循环或 `switch`；`continue` 结束当前这一轮循环，转到下一轮的推进过程。它们不负责撤销前面已经完成的修改，也不会自动结束整个方法。

## 完整例子

保存为 `BreakContinueDemo.java`，编译并运行；日志写到标准错误。

```java
import java.util.logging.Logger;

public class BreakContinueDemo {
    private static final Logger logger = Logger.getLogger(BreakContinueDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：控制单轮与整个循环
         * ========================================================================
         * 数据源：程序内准备的示例数据
         * 操作：
         * 1) 初始化输入
         * 2) 按控制流程处理并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 初始化累计值
        int sum = 0;
        // 1.2 跳过偶数，在遇到 5 时停止循环
        for (int n = 1; n <= 6; n++) {
            if (n == 5) {
                break;
            }
            if (n % 2 == 0) {
                continue;
            }
            sum += n;
        }
        // 1.3 输出已经累计的结果
        System.out.println("sum=" + sum);
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
sum=4
```

## 按顺序读代码

- `n=1`：不是 5，也不是偶数，累计 1。
- `n=2`：`n % 2 == 0` 为真，执行 `continue`，本轮不累计。对于普通 `for`，随后仍会执行更新表达式 `n++`。
- `n=3`：累计 3，当前和为 4；`n=4` 被跳过。
- `n=5`：先遇到 `break`，整个循环结束，6 不会被检查。

`continue` 在 `while` 中跳回条件检查，不会自动帮你推进计数器。如果计数器更新写在 `continue` 后面，就可能一直处理同一个值，造成死循环。

## 嵌套循环和工作用法

无标签的 `break` 只退出最内层循环；无标签的 `continue` 只跳过最内层当前轮次。标签可指定外层，但复杂逻辑通常更适合拆成方法并用 `return` 返回结果。业务处理前先想清楚：是跳过一条坏数据，还是停止整个批次；如果需要排查，也应记录原因。

## 手练与答案

把 `n == 5` 改成 `n == 7`：循环到 6 自然结束，累加 1、3、5，输出 `sum=9`。把 `continue` 改成 `break`：遇到 2 就退出，输出 `sum=1`。

课程原文：[廖雪峰 Java 教程：break和continue](https://liaoxuefeng.com/books/java/quick-start/flow/break-continue/index.html)。
