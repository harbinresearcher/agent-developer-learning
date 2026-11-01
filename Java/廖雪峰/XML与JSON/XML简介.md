# XML简介

> **重要程度：高**｜**手练：需要**
> **重点**：元素、属性、转义、文档结构。

## XML 是什么

XML 用标签组织有层次的数据。元素可以包含文本、属性和子元素；文档通常有一个根元素。解析或生成 XML 时应使用合适的解析器和写出器，而不是把外部字符串直接拼接进标签。

## 完整例子

保存为 `XmlIntroDemo.java`，编译运行；日志写到标准错误。

```java
import java.io.StringWriter;
import javax.xml.stream.XMLOutputFactory;
import javax.xml.stream.XMLStreamWriter;
import java.util.logging.Logger;
public class XmlIntroDemo {
    private static final Logger logger = Logger.getLogger(XmlIntroDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：构造示例 XML
         * ========================================================================
         * 数据源：固定的用户名文本
         * 操作：
         * 1) 创建根元素和名称元素
         * 2) 写入文本并读取结果
         */
        logger.info("开始构造 XML");
        // 1.1 创建字符输出并编写结构
        StringWriter buffer = new StringWriter();
        XMLStreamWriter writer = XMLOutputFactory.newFactory().createXMLStreamWriter(buffer);
        // 1.2 写入元素及需要转义的文本
        writer.writeStartElement("user");
        writer.writeStartElement("name");
        writer.writeCharacters("Ada & Bo");
        writer.writeEndElement();
        writer.writeEndElement();
        writer.close();
        System.out.println("xml=" + buffer.toString());
        logger.info("XML 构造完成");
    }
}
```

标准输出：

```text
xml=<user><name>Ada &amp; Bo</name></user>
```

## 执行过程

依次写入根元素 user、子元素 name 和文本 `Ada & Bo`。`writeCharacters` 在 XML 文本上下文中处理特殊字符，结果把 `&` 写成 `&amp;`；解析时应还原为原文本。结束两个元素后得到 `<user><name>Ada &amp; Bo</name></user>`。

## 结构与语义

元素必须正确嵌套并闭合，属性需要符合 XML 语法；同一个名字在不同命名空间中可能代表不同含义。XML 是否**格式良好**与是否符合业务约束或 XML Schema 是不同问题。显式 XML 声明和编码适用于字节表示；StringWriter 处理的是内存字符，本例没有写入文件字节。

## 工作中怎么用

读取外部 XML 时限制输入大小并配置解析器安全选项，禁止不需要的外部实体和 DTD；详情见 [[使用DOM]] 与 [[使用SAX]]。不同解析器选项并不完全通用，安全配置不支持时不应悄悄继续处理不可信输入。

## 手练与答案

将文本改为 `"A < B"`：写出器会对 XML 文本中的 `<` 做必要转义。把两个结束标签顺序写错，不再是正确嵌套的 XML；格式良好也不证明数据符合业务规则。

参考：[廖雪峰：XML简介](https://liaoxuefeng.com/books/java/xml-json/xml/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.xml/javax/xml/stream/XMLStreamWriter.html)。
