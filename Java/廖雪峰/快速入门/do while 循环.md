> **重要程度：中**｜**手练：建议**
> **重点**：先执行再判断。

而另一种`do while`循环则是先执行循环，再判断条件，条件满足时继续循环，条件不满足时退出。它的用法是：

```java
do {
    执行循环语句
} while (条件表达式);
```

可见，`do while`循环会至少循环一次。

我们把对1到100的求和用`do while`循环改写一下：

```java
// do-while
public class Main {
    public static void main(String[] args) {
        int sum = 0;
        int n = 1;
        do {
            sum = sum + n;
            n ++;
        } while (n <= 100);
        System.out.println(sum);
    }
}
```

使用`do while`循环时，同样要注意循环条件的判断。


### 小结

`do while`先执行循环，再判断条件；

`do while`循环会至少执行一次。

