> **重要程度：低**｜**手练：按需**
> **重点**：理解 JDK、JVM 和版本支持。

- JDK：Java Development Kit
- JRE：Java Runtime Environment
- 简单地说，JRE就是运行Java字节码的虚拟机。但是，如果只有Java源码，要编译成Java字节码，就需要JDK，因为JDK除了包含JRE，还提供了编译器、调试器等开发工具。

![[{9BE1DCC4-3331-4B4E-AB6D-E557B5E6887C}.png]]

- JSR规范：Java Specification Request
- JCP组织：Java Community Process
为了保证Java语言的规范性，SUN公司搞了一个JSR规范，凡是想给Java平台加一个功能，比如说访问数据库的功能，大家要先创建一个JSR规范，定义好接口

还要同时发布一个“参考实现”，以及一个“兼容性测试套件”：

- RI：Reference Implementation
- TCK：Technology Compatibility Kit
光有提议还不行，得贴出真正能跑的代码，这就是RI。

有其他人也想开发这样一个消息服务器，如何保证这些消息服务器对开发者来说接口、功能都是相同的？所以还得提供TCK。

