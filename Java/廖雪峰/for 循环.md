使用`for`循环时，千万不要在循环体内修改计数器！在循环体中修改计数器常常导致莫名其妙的逻辑错误。对于下面的代码：

```java
// for
public class Main {
    public static void main(String[] args) {
        int[] ns = { 1, 4, 9, 16, 25 };
        for (int i=0; i<ns.length; i++) {
            System.out.println(ns[i]);
            i = i + 1;
        }
    }
}
```

虽然不会报错，但是，数组元素只打印了一半，原因是循环内部的`i = i + 1`导致了计数器变量每次循环实际上加了`2`（因为`for`循环还会自动执行`i++`）。因此，在`for`循环中，不要修改计数器的值。


使用`for`循环时，计数器变量`i`要尽量定义在`for`循环中：

```java
int[] ns = { 1, 4, 9, 16, 25 };
for (int i=0; i<ns.length; i++) {
    System.out.println(ns[i]);
}
// 无法访问i
int n = i; // compile error!
```

如果变量`i`定义在`for`循环外：

```java
int[] ns = { 1, 4, 9, 16, 25 };
int i;
for (i=0; i<ns.length; i++) {
    System.out.println(ns[i]);
}
// 仍然可以使用i
int n = i;
```

那么，退出`for`循环后，变量`i`仍然可以被访问，这就破坏了变量应该把访问范围缩到最小的原则。

