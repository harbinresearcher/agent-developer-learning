# Class类

> **重要程度：中**｜**手练：建议**
> **重点**：类对象、获取方式和类加载。

## 核心概念

`Class` 对象表示运行时类型信息，可由类字面量、对象的 `getClass()` 或按名称加载获取。基本类型、数组和普通类也都有对应的 Class。

## 工作中注意

按名称加载可能触发类加载或初始化行为；反射操作需要处理找不到类和访问权限等异常。

## 手练

分别获取 String 类和字符串实例的 Class，比较两者；尝试加载不存在的类并观察异常。

来源：[教程：Class类](https://liaoxuefeng.com/books/java/reflection/class/index.html)。
