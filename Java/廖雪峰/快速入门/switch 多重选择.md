# switch 多重选择

> **重要程度：高**｜**手练：需要**
> **重点**：case 匹配、break、贯穿与版本区别。

## 先理解：一个值对应多种固定情况

订单状态是 0、1、2 等离散值时，`switch` 能把每个值对应的处理写在一起。它适合匹配固定值，不适合直接表达“分数大于 60”这样的范围；范围判断通常用 `if`。

## 完整例子：兼容 Java 11

保存为 `SwitchDemo.java`，编译并运行；标准输出如下，日志另写到标准错误。

```java
import java.util.logging.Logger;

public class SwitchDemo {
    private static final Logger logger = Logger.getLogger(SwitchDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：根据状态码选择动作
         * ========================================================================
         * 数据源：程序内准备的示例数据
         * 操作：
         * 1) 初始化输入
         * 2) 按控制流程处理并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备状态码
        int status = 1;
        // 1.2 匹配 case 并用 break 结束分支
        switch (status) {
            case 0:
                System.out.println("待处理");
                break;
            case 1:
                System.out.println("处理中");
                break;
            case 2:
                System.out.println("已完成");
                break;
            default:
                System.out.println("未知状态");
        }
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
处理中
```

## 按顺序读代码

`status` 为 1，`switch` 找到 `case 1`，输出“处理中”。`break` 退出整个 `switch`，所以不会执行 `case 2`。没有任何 case 匹配时，执行 `default`。`default` 可以放在其他位置，但一般放最后方便阅读。

**贯穿是旧语法的重要陷阱：** 删除 `case 1` 下面的 `break` 后，程序会继续执行后面的 `case 2`，输出“处理中”和“已完成”，直到遇到 `break`；它不会再次比较 `status`。多个 case 共用处理时可以有意贯穿，但应写明意图。

Java 11 的传统 `switch` 可用于 `byte`、`short`、`char`、`int` 及对应包装类、枚举和 `String`；不能直接用 `long` 或 `boolean`。包装类或字符串为 `null` 时，传统写法会抛出 `NullPointerException`，先检查空值。

## 版本区别与工作用法

Java 14 起，`switch` 表达式成为正式特性：可以用 `case ... ->` 表示匹配后执行且不贯穿，还可以让整个表达式返回值。Java 11 不能把它当正式语法使用；项目以实际 JDK 和编译配置为准。Java 21 扩展了模式匹配等能力，不要将其误当成 Java 11 的功能。项目中的业务状态建议用枚举表达，仍要处理外部传来的未知值。

## 手练与答案

1. 将 `status` 改成 3：输出“未知状态”。
2. 将 `status` 改成 0，同时去掉 `case 0` 后的 `break`：依次输出“待处理”“处理中”，然后在 `case 1` 的 `break` 处退出。

参考：[Oracle：switch 表达式](https://docs.oracle.com/en/java/javase/14/language/switch-expressions.html)、[Java 版本变化](https://docs.oracle.com/en/java/javase/21/language/java-language-changes-release.html)。

课程原文：[廖雪峰 Java 教程：switch 多重选择](https://liaoxuefeng.com/books/java/quick-start/flow/switch/index.html)。
