# 使用 JSON

> **重要程度：高**｜**手练：需要**
> **重点**：对象与数组、JSON 树、缺失/null、类型和业务校验。

## 解决什么问题

JSON 是常见的数据交换格式，包含对象、数组、字符串、数字、布尔值与 null。解析成功只说明输入符合解析器接受的语法，并不证明字段、类型和业务值都正确。
## 依赖与运行

创建 Maven 项目，将 Java 文件放入 `src/main/java`。以下完整 `pom.xml` 使用 Jackson 2.21.0，编译目标为 Java 11：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>example.notes</groupId><artifactId>json-demo</artifactId><version>1.0.0</version>
  <properties><maven.compiler.release>11</maven.compiler.release><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding></properties>
  <dependencies><dependency><groupId>com.fasterxml.jackson.core</groupId><artifactId>jackson-databind</artifactId><version>2.21.0</version></dependency></dependencies>
  <build><plugins><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.14.1</version></plugin></plugins></build>
</project>
```

在 pom 所在目录的 Windows PowerShell 中运行：

```powershell
mvn package
mvn org.apache.maven.plugins:maven-dependency-plugin:3.7.0:build-classpath "-Dmdep.outputFile=classpath.txt"
$notesDeps = (Get-Content -Raw -Encoding UTF8 classpath.txt).Trim()
```

随后运行本页示例类。首次运行可能下载依赖。本次使用 Maven 3.9.16、JDK 17 和 Java 11 编译目标构建，并用 Java 11 运行示例；示例没有配置自动测试，构建成功不代表所有 JSON 边界已测试。

## 完整例子：按树读取并校验

保存为 `src/main/java/JsonTreeDemo.java`：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.logging.Logger;
public class JsonTreeDemo {
    private static final Logger logger = Logger.getLogger(JsonTreeDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：解析并验证 JSON
         * ========================================================================
         * 数据源：固定的演示 JSON 文本
         * 操作：
         * 1) 解析 JSON 树并检查字段类型
         * 2) 输出经过检查的字段
         */
        logger.info("开始解析演示 JSON");
        // 1.1 解析固定输入并检查根节点及字段类型
        ObjectMapper mapper = new ObjectMapper();
        JsonNode root = mapper.readTree("{\"name\":\"Ada\",\"age\":18}");
        JsonNode name = root.path("name");
        JsonNode age = root.path("age");
        if (!root.isObject() || !name.isTextual() || !age.isIntegralNumber() || !age.canConvertToInt()) {
            throw new IllegalArgumentException("字段类型无效");
        }
        // 1.2 校验业务范围并输出
        int years = age.intValue();
        if (name.textValue().isBlank() || years < 0 || years > 120) {
            throw new IllegalArgumentException("字段值无效");
        }
        System.out.println("name=" + name.textValue());
        System.out.println("age=" + years);
        logger.info("演示 JSON 解析完成");
    }
}
```

运行 `java -cp "target/classes;$notesDeps" JsonTreeDemo`。标准输出：

```text
name=Ada
age=18
```

日志写到标准错误。本例使用固定的受控文本；真实输入必须限制大小并处理解析失败。

## 执行过程

`readTree` 将固定 JSON 解析为节点树；`path("name")` 和 `path("age")` 读取字段。示例检查根是对象、name 是字符串、age 是可转换为 int 的整数，再检查姓名非空白、年龄在示例范围内，最后输出字段。`path` 对缺失字段返回缺失节点，不应与显式 JSON null 混为一谈；本例两者都会因类型不符而被拒绝。

`readTree` 的空输入等边界可能返回 null；生产代码应先检查返回根节点再调用 `path`。数字可能超出 int 范围或需要精确小数，不能一律用 `asInt()` 静默取得默认值；先检查类型、范围和业务约定。

## 工作中怎么用

明确必填字段、未知字段、重复键、空值、日期格式、数字精度和错误响应。解析器可能有默认的类型转换或重复字段处理规则，按项目版本和安全要求配置并测试。不要把未验证的外部 JSON 直接当成可信对象，也不要输出密码或令牌。

限制输入大小和嵌套深度，并按 API 合理设置资源限制。JSON 本身不是 HTML 转义或 SQL 参数化；在不同输出边界要使用对应安全 API。

## 手练与答案

1. 删除 age、将 age 设为 null 或小数：示例类型检查失败并抛异常。
2. 将 age 设为 150：类型正确但业务范围检查失败，说明“可解析”与“业务有效”不同。

参考：[廖雪峰：使用JSON](https://liaoxuefeng.com/books/java/xml-json/json/index.html)；[Jackson 项目](https://github.com/FasterXML/jackson)。
