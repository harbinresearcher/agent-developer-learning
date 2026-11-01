# StringJoiner

> **重要程度：高**｜**手练：需要**
> **重点**：分隔符、前后缀、空集合。

## 解决什么问题

输出一组名称时，元素之间要有逗号，但开头和末尾不应多出分隔符。`StringJoiner` 自动处理分隔符，还可以设置前缀、后缀。

## 完整例子

保存为 `JoinerDemo.java`，用 `javac -encoding UTF-8 JoinerDemo.java` 编译，再运行 `java JoinerDemo`。日志写到标准错误。

```java
import java.util.StringJoiner;
import java.util.logging.Logger;
public class JoinerDemo {
    private static final Logger logger = Logger.getLogger(JoinerDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：构造带分隔符的文本
         * ========================================================================
         * 数据源：三个名字
         * 操作：
         * 1) 创建带前后缀的拼接器并逐项添加
         * 2) 输出拼接结果
         */
        logger.info("开始拼接名字");
        // 1.1 指定分隔符、前缀和后缀
        StringJoiner joiner = new StringJoiner(", ", "[", "]");
        // 1.2 添加元素并输出
        joiner.add("Ada").add("Lin").add("Ming");
        System.out.println(joiner.toString());
        logger.info("名字拼接完成");
    }
}
```

标准输出：

```text
[Ada, Lin, Ming]
```

## 代码和实践

创建时 `", "` 是分隔符，`"["` 和 `"]"` 是前后缀。每次 `add` 加入一个元素；`toString()` 在元素之间插入分隔符，得到 `[Ada, Lin, Ming]`。没有元素时，默认结果是前缀加后缀，即 `[]`；可用 `setEmptyValue` 显式更改空结果。

`StringJoiner` 是可变对象，`toString()` 得到 `String`。对已存在的字符串集合，`String.join(", ", names)` 或 Stream 的 `Collectors.joining` 也能处理类似需求；先选最直观的写法。`add(null)` 按 API 约定会添加文本 `null`，如果业务不希望如此，加入前应检查或过滤。

拼接出的文本不是自动安全的 SQL、HTML 或 JSON：特殊字符的编码、转义和参数化应由相应 API 完成。

## 手练与答案

1. 删除全部 `add`，结果为 `[]`。
2. 只加 `Ada`，结果为 `[Ada]`，不会在元素后多出逗号。

参考：[Oracle StringJoiner](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/StringJoiner.html)。

课程原文：[廖雪峰：StringJoiner](https://liaoxuefeng.com/books/java/oop/core/string-joiner/index.html)。
