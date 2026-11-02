# 编写 UI

> **重要程度：中**｜**手练：需要**
> **重点**：浏览器、控制器和模板如何协作；表单验证与安全转义；登录态、下单反馈与断线恢复。

## UI 在项目里负责什么

UI 向用户展示行情、订单和资产，接收用户操作，再请求服务端完成查询或提交。浏览器看到“已提交”不等于订单已成交；页面应显示服务端返回的明确状态，并允许用户查询订单。

教程用服务端 MVC 和 Pebble 模板渲染页面。下面用 Spring MVC 与 Thymeleaf 做一个独立表单练习，帮助理解同样的页面交互；它不是教程完整交易页面，也不实现登录或下单。

```text
浏览器 GET / → 控制器返回表单模板 → 浏览器展示
浏览器 POST /signup → 控制器校验 → 模板渲染结果 → 浏览器展示
```

## 手练：从页面提交表单

准备 JDK 17 与 Maven。在 `cloud-ui-demo` 目录创建四个文件：

`pom.xml`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version><relativePath/>
    </parent>
    <groupId>learn.example</groupId><artifactId>cloud-ui-demo</artifactId><version>1.0</version>
    <properties><java.version>17</java.version></properties>
    <dependencies>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
        <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-thymeleaf</artifactId></dependency>
    </dependencies>
    <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>
```

`src/main/java/learn/form/FormDemo.java`：

```java
package learn.form;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;
@SpringBootApplication
@Controller
public class FormDemo {
    private static final Logger logger = LoggerFactory.getLogger(FormDemo.class);
    @GetMapping("/")
    public String form() {
        /* ========================================================================
         * 步骤1：展示表单
         * 数据源：本地页面模板
         * 操作：1) 选择模板 2) 返回视图名称
         * ======================================================================== */
        logger.info("开始展示表单");
        // 1.1 返回模板视图
        String view = "form";
        logger.info("表单视图选择结束");
        return view;
    }
    @PostMapping("/signup")
    public ModelAndView signup(@RequestParam(name = "name", defaultValue = "") String name) {
        /* ========================================================================
         * 步骤1：校验并构造页面模型
         * 数据源：POST 表单中的 name 字段
         * 操作：1) 校验输入 2) 返回视图和状态
         * ======================================================================== */
        logger.info("开始处理表单");
        try {
            // 1.1 去除首尾空白，限制长度
            String cleaned = name.strip();
            ModelAndView result = new ModelAndView("result");
            if (cleaned.isEmpty() || cleaned.length() > 40) {
                result.setStatus(HttpStatus.BAD_REQUEST);
                result.addObject("message", "名称无效");
            } else {
                // 1.2 示例仅返回页面，不创建账户
                result.addObject("message", "你好，" + cleaned);
            }
            return result;
        } finally {
            logger.info("表单处理结束");
        }
    }
    public static void main(String[] args) {
        /* ========================================================================
         * 步骤1：启动表单示例
         * 数据源：MVC 配置和模板
         * 操作：1) 启动容器 2) 注册本地接口
         * ======================================================================== */
        logger.info("开始启动表单示例");
        // 1.1 启动本地 Web 服务
        SpringApplication.run(FormDemo.class, args);
        logger.info("表单示例启动完成");
    }
}
```

`src/main/resources/templates/form.html`：

```html
<!doctype html>
<html lang="zh">
<head><meta charset="UTF-8"><title>表单练习</title></head>
<body>
  <form method="post" action="/signup">
    <label for="name">名称</label>
    <input id="name" name="name" required maxlength="40">
    <button type="submit">提交</button>
  </form>
</body>
</html>
```

`src/main/resources/templates/result.html`：

```html
<!doctype html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
<head><meta charset="UTF-8"><title>结果</title></head>
<body><p th:text="${message}">结果</p></body>
</html>
```

在项目根目录运行：

```powershell
mvn package
java -jar target/cloud-ui-demo-1.0.jar --server.address=127.0.0.1 --server.port=18080
```

在浏览器打开 `http://127.0.0.1:18080/`，输入名称后提交。服务端会去掉首尾空白，合法名称显示“你好，名称”；空名称或超过 40 个字符返回 HTTP 400。浏览器的 `required` 与 `maxlength` 只能改善交互，服务端仍需校验。

本例已通过 HTTP 验证表单获取、有效和无效提交，以及输入 `<script>alert(1)</script>` 时 HTML 被转义；没有自动化验证浏览器实际点击。`th:text` 把数据作为文本输出，不能把用户输入当成可信 HTML 插入页面。Ctrl+C 可停止自己启动的服务。

## 页面与真实交易状态

1. **首次加载**：通过 API 查询当前资产、活动订单与最近行情；不要把前端缓存当作权威账本。
2. **提交订单**：生成并保留本次请求的稳定 ID；按钮禁用可以减少误触，但不能代替服务端幂等。
3. **等待结果**：区分提交中、已接受、部分成交、已成交、已撤销和失败等状态；接口超时后先查询或使用原请求 ID 重试。
4. **实时更新**：WebSocket 消息可更新界面；连接断开、漏消息或页面后台休眠后，要重新拉取权威快照并处理可能的重复或缺口。
5. **错误提示**：向用户说明可操作的下一步，不能展示内部堆栈、口令或令牌；涉及资金的操作还需明确确认和服务端风控。

## 登录和安全边界

教程登录成功后写入 Cookie。真实系统需使用 HTTPS、保护会话标识、按需求设置 `Secure`、`HttpOnly`、`SameSite`、过期时间和作用域；使用 Cookie 自动携带身份时，要结合框架提供的 CSRF 防护。权限检查必须在服务端完成。

跨域调用要按允许的来源、方法和凭证策略配置 CORS；CORS 不是授权机制，也不替代 CSRF 防护。页面模板应默认转义输出；插入 HTML、URL 或脚本时要按具体上下文处理。不要把真实口令或令牌写进前端源码、日志或示例。

本练习没有 Spring Security、登录态和 CSRF 保护，只演示表单与模板，不可改个地址就用于真实用户系统。

## 自测

1. 把浏览器的长度限制删掉，服务端为什么仍能拒绝超长名称？
2. 模板使用 `th:text` 时，用户输入的 `<script>` 为什么会显示成文本？这是否自动保护了所有输出上下文？
3. 用户点下单后请求超时，为什么不能直接提示“下单失败，可以重新下单”？
4. WebSocket 断线恢复后，如何确认页面上的订单状态没有漏更新？

来源：[原教程](https://liaoxuefeng.com/books/java/springcloud/ui/index.html)；[Spring MVC 表单](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods.html)。
