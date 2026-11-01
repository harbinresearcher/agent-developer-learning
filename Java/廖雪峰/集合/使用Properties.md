# 使用Properties

> **重要程度：高**｜**手练：需要**
> **重点**：字符串配置、默认值、编码与安全。

## 解决什么问题

`Properties` 保存键值配置，常用文本格式是每行一个 `key=value`。例如主机名、端口和超时可写在配置里，程序启动时读取，避免把环境差异硬编码到业务代码。

## 完整例子

保存为 `PropertiesDemo.java`，用 `javac -encoding UTF-8 PropertiesDemo.java` 编译，再运行 `java PropertiesDemo`。日志写到标准错误。

```java
import java.io.StringReader;
import java.util.Properties;
import java.util.logging.Logger;
public class PropertiesDemo {
    private static final Logger logger = Logger.getLogger(PropertiesDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：读取配置项
         * ========================================================================
         * 数据源：内存中的 properties 格式文本
         * 操作：
         * 1) 读取配置文本
         * 2) 获取已有项及缺省值
         */
        logger.info("开始读取配置");
        // 1.1 用字符读取器加载示例
        Properties props = new Properties();
        try (StringReader reader = new StringReader("host=localhost\nport=8080\n")) {
            props.load(reader);
        }
        // 1.2 输出配置和不存在时的默认值
        System.out.println("host=" + props.getProperty("host"));
        System.out.println("timeout=" + props.getProperty("timeout", "30"));
        logger.info("配置读取完成");
    }
}
```

标准输出：

```text
host=localhost
timeout=30
```

## 代码和实践

`load(reader)` 解析两行配置；`getProperty("host")` 得到字符串 `localhost`。没有 `timeout` 时，带第二个参数的 `getProperty` 返回默认字符串 `"30"`。配置项都是文本；需要整数时要解析并验证范围，不能把格式合法当成业务合法。

`load(Reader)` 接受已经解码的字符；读 UTF-8 文件时，应使用明确指定 UTF-8 的 Reader。`load(InputStream)` 按 properties 规范的 ISO-8859-1 字节规则解释，非对应字符可能需要转义；不要把两种重载的编码行为混为一谈。打开文件流或 reader 后应按其生命周期关闭。

properties 格式支持注释、转义和行续接；并非所有 `#`、反斜杠或空格都只是普通字符，应按格式规范处理。不要把密码、令牌放进版本库中的示例配置；生产配置按项目的密钥管理方式提供。

## 手练与答案

把文本改成包含 `timeout=10`：第二行输出 `timeout=10`。若端口配置成 `abc`，`Integer.parseInt` 会抛 `NumberFormatException`，调用方应提示配置错误。

参考：[Oracle：Properties](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Properties.html)。

课程原文：[廖雪峰：使用Properties](https://liaoxuefeng.com/books/java/collection/properties/index.html)。
