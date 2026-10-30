# JavaBean

> **重要程度：中**｜**手练：建议**
> **重点**：属性命名、getter/setter 约定、框架绑定与校验。

JavaBean 通常按约定通过属性访问器暴露对象数据。例如属性 `name` 对应 `getName()` 和 `setName(...)`；布尔属性可能使用 `is...`。一些框架依赖这类命名约定读取或设置属性。

框架是否需要无参构造器、是否支持不可变对象和 record，要看框架版本与具体配置。不要把“每个类都必须有公开 setter 和无参构造器”当作 Java 语言规则；业务约束仍需在合适边界校验。

手练：定义一个带姓名和年龄的对象，为年龄设置合法范围；说明直接反序列化外部输入后为什么仍需要验证。

来源：[教程：JavaBean](https://liaoxuefeng.com/books/java/oop/core/javabean/index.html)。
