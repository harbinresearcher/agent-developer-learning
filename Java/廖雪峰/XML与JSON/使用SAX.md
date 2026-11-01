# 使用SAX

> **重要程度：高**｜**手练：需要**
> **重点**：事件式解析、字符分片、DTD 安全。

## 解决什么问题

SAX 解析器按开始元素、字符、结束元素等事件通知处理器，不必像 DOM 那样保留整棵文档树。它适合顺序处理较大的 XML，但应用需要自己管理必要状态。

## 完整例子

保存为 `SaxDemo.java`，用 Java 11 或更高版本编译运行；日志写到标准错误。示例使用 JDK 默认解析器并拒绝 DOCTYPE。

```java
import java.io.StringReader;
import javax.xml.XMLConstants;
import javax.xml.parsers.SAXParserFactory;
import org.xml.sax.Attributes;
import org.xml.sax.InputSource;
import org.xml.sax.helpers.DefaultHandler;
import java.util.logging.Logger;
public class SaxDemo {
    private static final Logger logger = Logger.getLogger(SaxDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：按事件读取 XML
         * ========================================================================
         * 数据源：内存中的受控 XML 文本
         * 操作：
         * 1) 禁止 DTD 并创建 SAX 解析器
         * 2) 汇总 name 元素的字符内容
         */
        logger.info("开始读取 XML 事件");
        // 1.1 配置 JDK 默认 SAX 工厂
        SAXParserFactory factory = SAXParserFactory.newDefaultInstance();
        factory.setNamespaceAware(true);
        factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
        factory.setXIncludeAware(false);
        NameHandler handler = new NameHandler();
        // 1.2 解析固定输入并读取结果
        factory.newSAXParser().parse(new InputSource(new StringReader("<user><name>Ada</name></user>")), handler);
        System.out.println("name=" + handler.name);
        logger.info("XML 事件读取完成");
    }
    static class NameHandler extends DefaultHandler {
        private boolean insideName;
        private final StringBuilder name = new StringBuilder();
        public void startElement(String uri, String local, String qName, Attributes attributes) {
            if ("name".equals(local)) insideName = true;
        }
        public void characters(char[] chars, int start, int length) {
            if (insideName) name.append(chars, start, length);
        }
        public void endElement(String uri, String local, String qName) {
            if ("name".equals(local)) insideName = false;
        }
    }
}
```

标准输出：

```text
name=Ada
```

## 执行过程

遇到 name 开始标签，处理器设置 `insideName`；接收到字符事件时追加到 StringBuilder；遇到结束标签后停止追加。解析结束后输出 Ada。解析器可以把同一段文本拆成多次 `characters` 调用，所以不能假设一次回调就包含完整文本。

示例只为一个简单无命名空间的 name 元素服务；多个同名元素、嵌套 name、命名空间和缺失值都需要明确业务处理，不能把这个处理器当通用 XML 解析器。

## 安全与工作用法

示例配置拒绝 DTD、启用安全处理并禁用 XInclude；配置失败会向外抛异常，不应忽略后继续处理不可信输入。针对不同 parser、外部 schema 和转换操作还需分别配置外部访问限制，并限制输入与处理资源。SAX 是顺序事件，不会天然保证安全或数据有效。

按顺序处理时保存必要状态，避免把完整大文件不加限制地累积到内存。实际输入使用受控流并关闭资源；不要传入不可信 URL 让解析器自行访问。

## 手练与答案

把固定 name 改为 Bo：输出 name=Bo。若文档中有多个 name，当前处理器会把内容拼接，可能不符合业务；应为每条记录维护独立状态并验证结构。

参考：[廖雪峰：使用SAX](https://liaoxuefeng.com/books/java/xml-json/sax/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.xml/javax/xml/parsers/SAXParserFactory.html)。
