# JavaBean

> **重要程度：中**｜**手练：需要**
> **重点**：属性命名约定、getter/setter、无参构造、框架绑定的边界。

## 解决什么问题

JavaBean 是一组可被工具识别的 Java 类设计约定。框架和工具可以根据属性访问方法，读取或设置对象的数据。例如 `getName()` 和 `setName(...)` 表示一个名为 `name` 的属性。具体框架支持哪些构造方式、属性形式和可见性，应查该框架文档。

## 完整例子

保存为 `BeanDemo.java`，用 `javac -encoding UTF-8 BeanDemo.java` 编译，再运行 `java BeanDemo`；日志写到标准错误。

```java
import java.util.logging.Logger;
public class BeanDemo {
    private static final Logger logger = Logger.getLogger(BeanDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：创建并读取用户信息
         * ========================================================================
         * 数据源：演示用姓名
         * 操作：
         * 1) 创建对象并设置属性
         * 2) 使用 getter 读取属性
         */
        logger.info("开始读取用户信息");
        // 1.1 创建对象并设置姓名
        UserBean user = new UserBean();
        user.setName("Ada");
        // 1.2 通过 getter 读取属性
        System.out.println("name=" + user.getName());
        logger.info("用户信息读取完成");
    }
}
class UserBean {
    private String name;
    public UserBean() { }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

标准输出：`name=Ada`。

## 执行过程

`new UserBean()` 调用无参构造器，字段 `name` 此时默认为 null。`setName("Ada")` 把对象字段改为 Ada；`getName()` 返回这个字段。方法名对应可读、可写属性的常见命名约定。布尔属性还可能使用 `isActive()`；具体识别规则和框架支持范围需核对。

属性是对外访问的概念，字段是类的内部存储；一个计算得到的 getter 也可能代表可读属性，不要求一定有同名字段。只有 getter、没有 setter 的属性通常只读，但具体绑定行为依赖框架。

## 工作中怎么用

传统表单绑定、序列化和持久化框架可能使用 JavaBean 约定；不要假设所有框架都必须有公开无参构造器或都只能依赖 setter。新项目还可能使用构造器绑定、不可变对象或 record，取决于框架与 JDK 版本。

不必对每个字段机械添加公开 setter。需要保持业务不变量时，优先通过受控构造器或业务方法校验。外部输入绑定到对象后，仍需显式校验；getter/setter 不自动保证数据合法。

## 手练与答案

1. 不调用 `setName` 就读取：字段是 null，输出 `name=null`。
2. 将 `setName` 删除后，示例调用该方法无法编译；是否还能由某个框架绑定要看框架支持的其他构造或访问方式。

参考：[廖雪峰：JavaBean](https://liaoxuefeng.com/books/java/oop/core/javabean/index.html)；[Oracle JavaBeans 介绍](https://docs.oracle.com/javase/tutorial/javabeans/writing/properties.html)。
