# NullPointerException

> **重要程度：高**｜**手练：需要**
> **重点**：定位空引用、入口校验与缺失语义。

## 解决什么问题

引用为 `null` 表示没有指向对象。对它调用实例方法、访问实例字段，或发生需要拆箱的操作时可能抛 `NullPointerException`。定位时要先找出哪一个引用为 null，再追踪它为什么缺失。

## 完整例子

保存为 `NullDemo.java`，编译运行；日志写到标准错误。

```java
import java.util.Objects;
import java.util.logging.Logger;
public class NullDemo {
    private static final Logger logger = Logger.getLogger(NullDemo.class.getName());
    public static void main(String[] args) {
        /*
         * ========================================================================
         * 步骤1：检查必填姓名
         * ========================================================================
         * 数据源：缺失的姓名
         * 操作：
         * 1) 在方法入口校验引用
         * 2) 捕获并显示缺失原因
         */
        logger.info("开始检查姓名");
        // 1.1 传入 null 触发明确的入口校验
        try {
            nameLength(null);
        } catch (NullPointerException ex) {
            // 1.2 显示定位信息
            System.out.println("error=" + ex.getMessage());
        }
        logger.info("姓名检查完成");
    }
    static int nameLength(String name) {
        return Objects.requireNonNull(name, "name 不能为 null").length();
    }
}
```

默认运行的标准输出：

```text
error=name 不能为 null
```

## 执行过程

`nameLength(null)` 把 null 传给参数 `name`。`Objects.requireNonNull(name, "name 不能为 null")` 在该处立即抛出带明确消息的 `NullPointerException`，因此后面的 `.length()` 没有执行。调用方捕获后输出错误信息。

这里捕获异常是为了演示；真实程序应在合适边界把无效请求转成明确的错误响应或向上报告，不能为了消除异常就随意返回空字符串或 0。若 null 属于正常缺失状态，应在接口契约中说明并使用显式分支处理。

## 常见来源和排查

- `map.get(key)` 可能因键不存在返回 null，也可能因为该映射本身保存了 null；需要时用 `containsKey` 区分。
- 包装类型 `Integer` 为 null 时赋给 `int` 会触发拆箱异常。
- 链式调用 `a.getB().getC()` 中任何一步可能返回 null；根据堆栈、变量和实际运行环境定位，不盲目添加空判断。
- 有些 JDK 版本提供更详细的空指针提示，但消息内容依赖版本、编译和运行条件；不要把它当作稳定接口。

`Objects.equals(a, b)` 可在比较时处理 null，但不能替代对必填输入和业务约束的校验。

## 手练与答案

传入 `"Ada"` 时，方法返回 3，不进入 catch；示例不会输出这个返回值。去掉 `requireNonNull` 而直接 `name.length()`：传 null 时仍会抛 `NullPointerException`，但不再拥有本例显式设置的消息。

课程原文：[廖雪峰：NullPointerException](https://liaoxuefeng.com/books/java/exception/null-pointer-exception/index.html)。
