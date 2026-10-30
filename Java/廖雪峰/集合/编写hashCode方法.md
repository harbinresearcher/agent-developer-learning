# 编写 hashCode 方法

> **重要程度：高**｜**手练：需要**
> **重点**：记住 equals 与 hashCode 的契约；解释哈希冲突；理解可变键为什么危险。

## 先看问题：HashMap 怎样找键

假设用坐标对象当 Map 的键。查找时，哈希映射先利用键的哈希值定位候选位置，再根据键的相等规则确定要找的对象。哈希用于缩小查找范围，不是对象身份编号，也不是不会重复的唯一值。

因此，同样代表 `(1, 2)` 的两个坐标对象，如果 `equals` 判断它们相等，就必须返回相同的 `hashCode`。否则哈希集合可能在不相关的位置查找，导致逻辑相等的对象未被识别。

## 最重要的两条关系

- 如果 `a.equals(b)` 为 true，那么 `a.hashCode() == b.hashCode()` **必须为 true**。
- 如果两个 hashCode 相同，`a.equals(b)` **不一定为 true**。不同对象可能发生哈希冲突，集合还需要根据 equals 判断。

在同一程序运行期间，如果参与 equals 判断的对象信息没有改变，重复调用 hashCode 应保持一致。不要把 Java 的 `hashCode()` 当成密码哈希或安全摘要。

## 实例：两个相同坐标为什么只留一个

先运行 [[编写equals方法]] 中的 `EqualityDemo.java`：它创建两个不同的 Point 实例，都表示 `(1, 2)`；`equals` 比较 x、y，`hashCode` 也根据 x、y 计算。

按代码顺序看：

1. 两次 `new` 得到不同对象，引用比较结果为 false。
2. 坐标字段相同，equals 返回 true。
3. 计算 hashCode 时使用与相等判断一致的字段，因此两个相等对象的哈希值一致。
4. 两个对象加入 HashSet，集合按契约识别重复，最终数量为 1。

`Objects.hash(x, y)` 是实现方式之一；不要求自己设计复杂算法。更重要的是：**equals 使用哪些字段，hashCode 就要与相等规则保持一致**。

## 为什么不建议使用可变键

假设对象放入 HashMap 后，又修改了参与 equals / hashCode 的字段。它的新哈希值可能不同，但 Map 中该条目当初按旧值放置。之后再拿同一个对象查找，也可能找不到预期记录。

优先使用不可变键，或者保证作为键期间相等相关字段不变。业务对象是否要按数据库 ID、全部字段或其他内容比较，要按实际业务契约设计。

## 手练与自测

1. 运行 [[编写equals方法]] 的例子，把第二个坐标改成 `(1, 3)`，检查集合数量。
2. 两个对象 hashCode 相同，能否直接断定 equals 为 true？
3. 两个对象 equals 为 true，但 hashCode 不同，会破坏什么约定？
4. 为什么在 HashMap 中修改键的相等字段有风险？

**答案要点**：① 两个不同坐标应分别保留。② 不能，可能发生哈希冲突。③ 违反相等对象必须有相同哈希的契约。④ 对象插入位置与修改后的哈希值可能不再匹配。

参考：[教程：编写hashCode方法](https://liaoxuefeng.com/books/java/collection/hashcode/index.html)；[Java Object API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html)。
