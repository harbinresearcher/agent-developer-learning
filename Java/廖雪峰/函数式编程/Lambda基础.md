# Lambda基础

> **重要程度：高**｜**手练：需要**
> **重点**：函数式接口和变量捕获。

## 核心概念

Lambda 可实现只有一个抽象方法的函数式接口，使简单行为能作为参数传递。

## 工作中注意

捕获的局部变量必须是 final 或事实上的 final；捕获对象引用不代表对象状态不可变。

## 手练

将一个排序规则写成 Lambda，再写成普通实现，比较可读性。

来源：[教程：Lambda基础](https://liaoxuefeng.com/books/java/functional/basic/index.html)。

## 关联知识与实践

函数式接口只有一个抽象方法，lambda 可作为该接口的实现；方法引用是适用时更简洁的写法。捕获的局部变量必须是 final 或事实上的 final，避免把它与对象内部状态是否可变混为一谈。

来源：[教程：函数式编程](https://liaoxuefeng.com/books/java/functional/index.html)；[Java Stream API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)。
