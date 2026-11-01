# Class类

> **重要程度：中**｜**手练：建议**
> **重点**：Class 对象、获取方式、加载与初始化。

## 解决什么问题

通常代码在编译时已经知道类型。框架、测试工具或插件系统有时要在运行时查看类型的名称、构造器、字段或方法，`Class` 对象就是这些元信息的入口。它描述一个运行时类型，不是该类型的普通业务实例。

## 完整例子

保存为 `ClassInfoDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.logging.Logger;
public class ClassInfoDemo {
    private static final Logger logger = Logger.getLogger(ClassInfoDemo.class.getName());
    public static void main(String[] args) throws ClassNotFoundException {
        /*
         * ========================================================================
         * 步骤1：获取和比较运行时类型
         * ========================================================================
         * 数据源：String 类字面量、实例与类名
         * 操作：
         * 1) 用三种方式取得 Class 对象
         * 2) 比较对象并读取类型名
         */
        logger.info("开始读取类型信息");
        // 1.1 获取 String 的运行时类型
        Class<String> byLiteral = String.class;
        Class<?> byObject = "hello".getClass();
        Class<?> byName = Class.forName("java.lang.String");
        // 1.2 比较 Class 对象并显示名称
        System.out.println("literal==object: " + (byLiteral == byObject));
        System.out.println("literal==name: " + (byLiteral == byName));
        System.out.println("name: " + byName.getName());
        logger.info("类型信息读取完成");
    }
}
```

标准输出：

```text
literal==object: true
literal==name: true
name: java.lang.String
```

## 执行过程

`String.class` 使用类字面量；`"hello".getClass()` 从已有对象取得它的实际运行时类型；`Class.forName("java.lang.String")` 使用完整类名查找。它们在本例指向同一个 String 类型的 `Class` 对象，所以两次引用比较为 true。`Class<?>` 表示具体类型参数在此不预先指定。

## 重要边界

`Class.forName(String)` 会按其 API 规则加载并初始化目标类；类找不到会抛受检异常 `ClassNotFoundException`，初始化失败还可能产生其他错误。需要控制初始化时可研究带 `initialize` 和类加载器参数的重载。类加载、链接和初始化是不同阶段，不要把它们当成同一件事。

相同类名不一定保证相同运行时类型：类加载器参与类身份，不同加载器定义的同名类可能不是同一个类型。基本类型及数组也有 `Class`，例如 `int.class` 和 `String[].class`；`getClass()` 不能对 null 调用。不能用 `Class.forName("int")` 获取 `int.class`。

## 工作中怎么用

优先用编译期类型和接口；框架确需动态发现类型时再使用反射，并明确来源、允许加载的类型和失败处理。类名来自不可信输入时，不应任意加载并初始化未知类。泛型对象的 `Class` 通常不能区分所有实际类型参数，详见 [[泛型和反射]]。

## 手练与答案

1. `String.class == "x".getClass()` 为 true。
2. `Class.forName("不存在的完整类名")` 通常抛 `ClassNotFoundException`；本例的 `main throws` 会让异常传播而不是给出友好提示。

参考：[廖雪峰：Class类](https://liaoxuefeng.com/books/java/reflection/class/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Class.html)。
