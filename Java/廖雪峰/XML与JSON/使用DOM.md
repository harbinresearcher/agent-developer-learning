# 使用DOM

> **重要程度：高**｜**手练：需要**
> **重点**：文档树、选择元素、禁止 DTD 和外部实体。

## 解决什么问题

DOM 将 XML 解析成内存中的文档树，便于按节点访问和修改；代价是需要为整棵树占用内存。处理外部 XML 时必须先限制资源和外部实体行为。

## 完整例子

保存为 `DomDemo.java`，用 Java 11 或更高版本编译运行；日志写到标准错误。示例选择 JDK 内置解析器并拒绝所有 DOCTYPE。

```java
import java.io.StringReader;
import javax.xml.XMLConstants;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;
import org.xml.sax.SAXParseException;
import org.xml.sax.helpers.DefaultHandler;
import java.util.logging.Logger;
public class DomDemo {
    private static final Logger logger = Logger.getLogger(DomDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：安全地解析受控 XML
         * ========================================================================
         * 数据源：内存中的示例 XML
         * 操作：
         * 1) 配置禁止 DTD 和外部访问
         * 2) 读取元素并验证 DTD 被拒绝
         */
        logger.info("开始解析 XML");
        // 1.1 创建并安全配置 JDK 内置 DOM 工厂
        DocumentBuilderFactory factory = DocumentBuilderFactory.newDefaultInstance();
        factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
        factory.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
        factory.setAttribute(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
        factory.setXIncludeAware(false);
        DocumentBuilder builder = factory.newDocumentBuilder();
        builder.setErrorHandler(new DefaultHandler() {
            public void fatalError(SAXParseException ex) throws SAXException { throw ex; }
        });
        // 1.2 解析正常文本，并检查拒绝 DTD
        Document doc = builder.parse(new InputSource(new StringReader("<user><name>Ada</name></user>")));
        System.out.println("name=" + doc.getElementsByTagName("name").item(0).getTextContent());
        try {
            builder.parse(new InputSource(new StringReader("<!DOCTYPE user><user/>")));
            System.out.println("doctypeRejected=false");
        } catch (SAXException ex) {
            System.out.println("doctypeRejected=true");
        }
        logger.info("XML 解析完成");
    }
}
```

标准输出：

```text
name=Ada
doctypeRejected=true
```

## 执行过程

工厂使用 `newDefaultInstance()` 选择 JDK 默认实现；示例开启安全处理，拒绝 DOCTYPE，禁止外部 DTD 和 Schema 访问，并禁用 XInclude。解析固定文本后，取第一个 name 元素得到 Ada。第二段输入包含 DOCTYPE，解析抛 SAXException，示例捕获并显示它被拒绝。

`getElementsByTagName(...).item(0)` 在没有元素时可能返回 null；真实输入应验证根元素、命名空间、元素数量与值，不能假定格式和业务数据都正确。真实文档可能包含多个匹配元素，简单取第一个可能掩盖错误。

## 安全与工作用法

XXE 可能导致读取本地文件、对外请求或资源耗尽。针对不可信 XML，优先拒绝不需要的 DTD；限制输入大小和解析资源。若所需安全配置不受当前 parser 支持，应该中止并排查，而不是捕获配置异常后以不安全模式继续。`setExpandEntityReferences(false)` 不是禁止外部访问的充分替代。

这里拒绝 DOCTYPE 会使确实依赖 DTD 的文档不能解析；若业务必须使用 DTD，需要按所用实现配置外部访问、实体解析器及资源上限。DOM 会构建整棵树，大文件应评估流式解析。

## 手练与答案

将 name 改为 Bo：输出 name=Bo。将正常 XML 删除 name 元素：示例后续取第 0 个元素会失败，实际程序应显式校验缺失。示例拒绝 DOCTYPE 不等同于完成所有业务校验。

参考：[OWASP XXE 防护](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)。

参考：[廖雪峰：使用DOM](https://liaoxuefeng.com/books/java/xml-json/dom/index.html)；[Oracle API](https://docs.oracle.com/en/java/javase/17/docs/api/java.xml/javax/xml/parsers/DocumentBuilderFactory.html)。
