> **重要程度：高**｜**手练：需要**
> **重点**：main 入口、编译与运行。

`public`、`class`都是Java的关键字，必须小写，`Hello`是类的名字，按照习惯，首字母`H`要大写

`public`、`static`用来修饰方法，这里表示它是一个公开的静态方法，`void`是方法的返回类型,方法的代码每一行用`;`结束


某个类定义的`public static void main(String[] args)`是Java程序的固定入口方法，因此，Java程序总是从`main`方法开始执行

注意到Java源码的缩进不是必须的，但是用缩进后，格式好看

文件名必须是`Hello.java`，而且文件名也要注意大小写，因为要和我们定义的类名`Hello`完全保持一致。

需要先用`javac`把`Hello.java`编译成字节码文件`Hello.class`，然后，用`java`命令执行这个字节码文件![[{6CBEFCCB-45C3-4F6A-A1F9-A2C233594EA0}.png]]


### 小结

一个Java源码只能定义一个`public`类型的class，并且class名称和文件名要完全一致；

使用`javac`可以将`.java`源码编译成`.class`字节码；

使用`java`可以运行一个已编译的Java程序，参数是类名。