# 编写 equals 方法

> **重要程度：高**｜**手练：需要**
> **重点**：区分同一个对象和逻辑相等；掌握 equals 的基本契约；理解它与 hashCode 的关系。

## 先看问题

两个坐标对象分别通过 `new` 创建，但坐标都是 `(1, 2)`。它们不是同一个对象，却可能表示同一个位置。`==` 比较两个引用是否指向同一对象；我们可以用 `equals` 定义对象的逻辑相等。

## 完整例子

保存为 `EqualityDemo.java`；运行 `javac -encoding UTF-8 EqualityDemo.java`，再运行 `java EqualityDemo`。
```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;
import java.util.logging.Logger;

public class EqualityDemo {
    private static final Logger logger = Logger.getLogger(EqualityDemo.class.getName());

    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：比较值对象并放入集合
         * ========================================================================
         * 目标：验证 equals 与 hashCode 的配合。
         * 数据源：两个坐标相同的对象。
         * 操作：
         * 1) 比较两个实例是否逻辑相等。
         * 2) 将它们放入 HashSet 并检查数量。
         */
        logger.info("开始比较坐标");
        // 1.1 创建并比较两个坐标对象
        Point first = new Point(1, 2);
        Point second = new Point(1, 2);
        System.out.println("同一对象: " + (first == second));
        System.out.println("内容相等: " + first.equals(second));
        // 1.2 通过集合检查去重结果
        Set<Point> points = new HashSet<>();
        points.add(first);
        points.add(second);
        System.out.println("集合数量: " + points.size());
        logger.info("坐标比较完成");
    }

    static final class Point {
        private final int x;
        private final int y;

        Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public boolean equals(Object other) {
            if (this == other) return true;
            if (!(other instanceof Point)) return false;
            Point point = (Point) other;
            return x == point.x && y == point.y;
        }

        @Override
        public int hashCode() {
            return Objects.hash(x, y);
        }
    }
}
```

标准输出依次为 `同一对象: false`、`内容相等: true`、`集合数量: 1`。

### 看执行结果

1. 两次 `new Point(1, 2)` 创建了两个对象，因此 `first == second` 为 false。
2. `equals` 先处理相同引用，再判断对方是否为 Point，然后比较 x 与 y；坐标相同，因此返回 true。
3. 将两个对象加入 HashSet 时，集合依据哈希和相等规则判断它们是重复值，因此数量为 1。两字段是 `final`，示例类也设为 final，避免这个例子还要处理可变键和子类相等语义。

## equals 要满足什么

对非 null 对象，常见契约包括：自反（自己等于自己）、对称（a 等于 b 时 b 也等于 a）、传递、一致；与 null 比较应返回 false。写法可用 `@Override` 检查方法签名是否正确。

**只重写 equals 不够。** 如果两个对象相等，`hashCode()` 必须相同，否则 HashSet 和 HashMap 可能无法按预期找到它们。反过来，hashCode 相同并不代表对象必然相等；哈希冲突可以存在。详见 [[编写hashCode方法]]。

## 手练与自测

1. 把第二个坐标改成 `(1, 3)`，预测三行输出。
2. 为什么 `==` 为 false，`equals` 可以为 true？
3. 如果重写 equals，为什么还需要同步考虑 hashCode？

**答案要点**：① false、false、2。② 两个引用指向不同实例，但业务字段相等。③ 相等对象必须有相同哈希，哈希集合依赖该契约。

参考：[教程：编写equals方法](https://liaoxuefeng.com/books/java/collection/equals/index.html)；[Java Object API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html)。
