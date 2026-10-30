# Java的异常

> **重要程度：高**｜**手练：需要**
> **重点**：异常体系、受检异常和调用栈。

## 核心概念

Java 使用异常对象报告无法正常完成的情况。异常沿调用栈向上传播，直到被匹配的处理器捕获；未处理时，当前线程可能终止。`Throwable` 下分为 `Exception` 与 `Error`，应用通常针对可处理的异常类型编写处理逻辑。

## 工作中注意

`RuntimeException` 及其子类属于非受检异常；其他多数 `Exception` 子类属于受检异常，编译器要求捕获或声明。错误分类不等于能否恢复，仍需按具体场景判断。

## 手练

制造一次整数解析失败，先读异常类型和消息，再从调用栈定位自己的代码；说明 `NumberFormatException` 属于哪一类。

来源：[教程：Java的异常](https://liaoxuefeng.com/books/java/exception/java-exception/index.html)。
