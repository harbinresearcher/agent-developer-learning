# if条件判断

> **重要程度：高**｜**手练：需要**
> **重点**：条件顺序、边界、字符串比较。

## 先理解：为什么需要分支

同一段程序要根据不同输入做不同处理。分数为 80 时，应显示及格；若为 101，则应先拦截非法输入。`if` 后的括号里必须是 `boolean` 表达式，只有结果为 `true` 才进入对应代码块。

## 完整例子

保存为 `IfDemo.java`，用 `javac -encoding UTF-8 IfDemo.java` 编译，再用 `java IfDemo` 运行。日志写到标准错误，下面单独展示标准输出。

```java
import java.util.logging.Logger;

public class IfDemo {
    private static final Logger logger = Logger.getLogger(IfDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：按分数选择分支
         * ========================================================================
         * 数据源：程序内准备的示例数据
         * 操作：
         * 1) 初始化输入
         * 2) 按控制流程处理并输出结果
         */
        logger.info("开始处理示例");
        // 1.1 准备待判断的分数
        int score = 80;
        // 1.2 从上往下选择第一个满足条件的分支
        if (score < 0 || score > 100) {
            System.out.println("分数无效");
        } else if (score >= 90) {
            System.out.println("优秀");
        } else if (score >= 60) {
            System.out.println("及格");
        } else {
            System.out.println("不及格");
        }
        logger.info("示例处理完成");
    }
}
```

标准输出：

```text
及格
```

## 按顺序读代码

1. `score` 的值是 80。第一项 `score < 0 || score > 100` 为假，跳过非法分数分支。
2. `score >= 90` 为假，继续检查 `score >= 60`，结果为真，打印“及格”。
3. 这一组 `if / else if / else` 最多只执行一个分支；一旦命中，后面的条件不会再检查。最后的 `else` 捕获前面都没命中的情况，也可以省略。

**顺序决定结果。** 若把 `score >= 60` 放在 `score >= 90` 前，95 会先命中及格，永远进不了优秀分支。相邻边界至少测试 -1、0、59、60、89、90、100、101。

`==` 比较基本类型的值；对于引用，`==` 判断是否指向同一对象。比较字符串内容用 `"ADMIN".equals(role)`：常量放前面时，`role` 为 `null` 会返回 `false`。不要把 `role.equals("ADMIN")` 当成能处理 `null` 的写法。

## 工作中怎么用

先拒绝无效参数，再按范围处理；多个条件组合用 `&&`（同时满足）、`||`（至少一个满足），并用括号表达意图。分支很多时可拆成有名字的方法，避免嵌套过深。即使只有一行，也保留花括号，降低后续修改出错的概率。

## 手练与答案

把分数改为 90、59、101，分别预测输出再运行。答案依次是“优秀”“不及格”“分数无效”。为什么要先检查 101？否则它满足 `>= 90`，会被误判为优秀。

课程原文：[廖雪峰 Java 教程：if条件判断](https://liaoxuefeng.com/books/java/quick-start/flow/if/index.html)。
