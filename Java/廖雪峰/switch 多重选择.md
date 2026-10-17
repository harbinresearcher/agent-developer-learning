除了`if`语句外，还有一种条件判断，是根据某个表达式的结果，分别去执行不同的分支。

例如，在游戏中，让用户选择选项：

1. 单人模式
2. 多人模式
3. 退出游戏

这时，`switch`语句就派上用场了。


我们看一个例子：

```java
// switch
public class Main {
    public static void main(String[] args) {
        int option = 1;
        switch (option) {
        case 1:
            System.out.println("Selected 1");
            break;
        case 2:
            System.out.println("Selected 2");
            break;
        case 3:
            System.out.println("Selected 3");
            break;
        }
    }
}
```

如果`option`的值没有匹配到任何`case`，例如`option = 99`，那么，`switch`语句不会执行任何语句。这时，可以给`switch`语句加一个`default`，当没有匹配到任何`case`时，执行`default`：
```java
default:
            System.out.println("Selected other");
            break;
```


如果有几个`case`语句执行的是同一组语句块，可以这么写：

```java
// switch
public class Main {
    public static void main(String[] args) {
        int option = 2;
        switch (option) {
        case 1:
            System.out.println("Selected 1");
            break;
        case 2:
        case 3:
            System.out.println("Selected 2, 3");
            break;
        default:
            System.out.println("Selected other");
            break;
        }
    }
}
```

`switch`语句还可以匹配字符串。字符串匹配时，是比较“内容相等”。例如：

```java
// switch
public class Main {
    public static void main(String[] args) {
        String fruit = "apple";
        switch (fruit) {
        case "apple":
            System.out.println("Selected apple");
            break;
```

