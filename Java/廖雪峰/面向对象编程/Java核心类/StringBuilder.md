# StringBuilder

> **重要程度：高**｜**手练：需要**
> **重点**：可变拼接、toString 快照和线程边界。

## 先理解：反复拼接文本

`String` 不可变；需要按循环逐段构造一段较长文本时，`StringBuilder` 提供可修改的字符序列。调用 `append` 改变构建器自身，最后用 `toString()` 得到 `String`。

## 完整例子

保存为 `BuilderDemo.java`，编译并运行；日志写到标准错误。

```java
import java.util.logging.Logger;

public class BuilderDemo {
    private static final Logger logger = Logger.getLogger(BuilderDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：构建并修改文本
         * ========================================================================
         * 数据源：三段文本
         * 操作：
         * 1) 拼接并保存字符串结果
         * 2) 继续修改构建器并比较结果
         */
        logger.info("开始拼接文本");
        // 1.1 逐段添加并生成字符串
        StringBuilder builder = new StringBuilder();
        builder.append("Java").append(" ").append("入门");
        String first = builder.toString();
        // 1.2 继续修改构建器
        builder.append("笔记");
        System.out.println("first=" + first);
        System.out.println("current=" + builder.toString());
        logger.info("文本拼接完成");
    }
}
```

标准输出：

```text
first=Java 入门
current=Java 入门笔记
```

## 按顺序理解

新建构建器时内容为空。三次 `append` 后，它的内容为 `Java 入门`；`append` 返回构建器本身，因此可以链式调用。`toString()` 返回一个字符串，赋给 `first`。之后再向构建器追加“笔记”，`first` 仍保持原来的内容，第二次 `toString()` 返回修改后的内容。

`length()` 是当前 UTF-16 代码单元的数量，`capacity()` 是当前容量，两者不是一回事。常用操作还有 `insert`、`delete`、`replace`；下标和范围错误会抛异常，尤其要注意某些操作的结束位置是排他的。

## 工作中怎么用

当片段数量由循环或运行时数据决定时，构建器有助于避免在循环中反复生成中间字符串。少量固定字符串用 `+` 通常更清楚；编译器和运行时可能优化拼接，性能结论应靠实际测量。带分隔符拼接可考虑 `StringJoiner`。

`StringBuilder` 不提供线程安全保证：多个线程共享修改需要外部同步或其他设计。将它转成 `String` 后可以传递不可变结果，但这不让其他共享状态自动线程安全。

## 手练与答案

1. 把最后的追加改成 `builder.append("!")`：`first` 保持 `Java 入门`，`current` 为 `Java 入门!`。
2. 在首次 `toString()` 前追加“笔记”：两行都包含“笔记”。

参考：[Oracle：StringBuilder](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/StringBuilder.html)。

课程原文：[廖雪峰 Java 教程：StringBuilder](https://liaoxuefeng.com/books/java/oop/core/string-builder/index.html)。
