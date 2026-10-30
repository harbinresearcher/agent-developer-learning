# super通配符

> **重要程度：高**｜**手练：需要**
> **重点**：下界通配符、写入和读取类型。

## 核心概念

`? super Integer` 表示 Integer 或其某个父类型。方法向集合写入整数时，可以接收 `List<Integer>`、`List<Number>` 或 `List<Object>`。

## 工作中注意

读取时编译器无法知道元素具体是哪个父类型，因此通常只能安全地按 Object 使用。生产者用 extends、消费者用 super 是判断方向的口诀，仍需结合实际数据流。

## 手练

写一个向集合添加整数的方法；分别传入 Integer、Number 和 Object 列表，解释读取时为什么不能直接当 Integer。

来源：[教程：super通配符](https://liaoxuefeng.com/books/java/generics/super/index.html)。
