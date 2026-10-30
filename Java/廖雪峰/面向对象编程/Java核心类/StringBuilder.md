# StringBuilder

> **重要程度：高**｜**手练：需要**
> **重点**：可变追加、`append`、`toString` 和线程边界。

`StringBuilder` 维护可变的字符序列，适合循环中逐步拼接字符串；每次 `append` 通常返回同一个构建器，最后用 `toString()` 得到结果字符串。不要把它理解成会修改已有的 `String` 对象。

简单的一次字符串拼接无需机械改成构建器；编译器可能采用不同优化。`StringBuilder` 不是线程安全的共享容器，多线程使用应避免共享或自行同步。

手练：把一组姓名按行追加到构建器，最后输出；比较在循环中反复拼接不可变字符串的写法，并用测量而不是猜测评估性能。

来源：[教程：StringBuilder](https://liaoxuefeng.com/books/java/oop/core/string-builder/index.html)。
