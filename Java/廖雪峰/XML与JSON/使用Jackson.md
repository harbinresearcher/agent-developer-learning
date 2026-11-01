# 使用 Jackson

> **重要程度：高**｜**手练：需要**
> **重点**：对象绑定、序列化往返、版本兼容和输入验证。

## 解决什么问题

Jackson 可把 JSON 映射为 Java 对象，也可把对象转换成 JSON。对象绑定减少手写字段处理，但不会自动保证输入符合业务约束，输出字段也可能包含不该暴露的信息。
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

## 完整例子：绑定对象并往返

保存为 `src/main/java/JacksonBindingDemo.java`：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.logging.Logger;
public class JacksonBindingDemo {
    private static final Logger logger = Logger.getLogger(JacksonBindingDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：绑定和往返演示数据
         * ========================================================================
         * 数据源：固定的用户 JSON
         * 操作：
         * 1) 绑定对象并检查业务字段
         * 2) 序列化、反序列化并读取结果
         */
        logger.info("开始绑定演示 JSON");
        // 1.1 将 JSON 绑定为对象并做业务校验
        ObjectMapper mapper = new ObjectMapper();
        User user = mapper.readValue("{\"name\":\"Ada\",\"age\":18}", User.class);
        if (user.name == null || user.name.isBlank() || user.age == null || user.age < 0 || user.age > 120) {
            throw new IllegalArgumentException("用户数据无效");
        }
        // 1.2 序列化后再读取
        String json = mapper.writeValueAsString(user);
        User restored = mapper.readValue(json, User.class);
        System.out.println("name=" + restored.name);
        System.out.println("age=" + restored.age);
        logger.info("演示 JSON 绑定完成");
    }
    public static class User {
        public String name;
        public Integer age;
        public User() { }
    }
}
```

运行 `java -cp "target/classes;$notesDeps" JacksonBindingDemo`。标准输出：

```text
name=Ada
age=18
```

日志写到标准错误。示例中的 User 是为理解绑定而使用的简单数据对象，不代表生产对象应公开所有字段。

## 执行过程

`readValue` 把 JSON 映射到 User；age 使用包装类型，使缺失值可以与 0 区分。代码检查 name 非空白、age 存在且在示例范围；`writeValueAsString` 产生 JSON，再由同一配置读回并输出。示例不依赖输出字段的顺序。

Jackson 的类型转换、未知字段和其他反序列化行为受版本与配置影响；本例的值校验不能证明所有输入类型都被严格拒绝。例如字符串是否可转换为数字要按具体配置验证。对需要严格格式的接口，可先检查 JSON 节点类型或使用合适的校验配置和测试。

## 版本与工作用法

本页是 Jackson 2.x，包名为 `com.fasterxml.jackson`。Jackson 3.x 有不同的主要包名、依赖坐标和版本要求，不能只改版本号就假定完全兼容；以项目依赖与官方迁移文档为准。`ObjectMapper` 通常应在使用前完成配置并复用，不要在并发使用中随意更改配置。

将外部输入映射到明确 DTO，再验证必填、范围和业务关系；响应也使用明确输出模型，避免把密码、令牌或内部字段直接序列化。谨慎处理多态类型和不可信输入，不要随意开启危险的自动类型绑定；限制输入大小并更新依赖。

## 手练与答案

1. age 改为 null 或删除：示例校验抛 `IllegalArgumentException`。
2. age 改为 150：虽然 JSON 格式正确，仍因业务范围而被拒绝。
3. 如果向 User 新增公开的敏感字段，是否一定不会输出？不能保证；应通过输出 DTO 和测试明确控制序列化内容。

参考：[廖雪峰：使用Jackson](https://liaoxuefeng.com/books/java/xml-json/jackson/index.html)；[Jackson 3 迁移指南](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md)。
