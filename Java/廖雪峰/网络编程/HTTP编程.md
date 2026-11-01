# HTTP 编程

> **重要程度：高**｜**手练：需要**
> **重点**：请求与响应、Java 11 HttpClient、超时、状态码、幂等与资源边界。

## 先理解请求和响应

HTTP 客户端发送请求方法、URL、头部和可选正文；服务端返回状态码、头部和正文。状态码表示 HTTP 层结果，业务是否成功还要按接口约定检查响应内容。Java 11 提供标准 `java.net.http.HttpClient`。

## 完整例子：只访问本机

保存为 `HttpLoopbackDemo.java`。本例使用 JDK 的 `jdk.httpserver` 模块创建本机测试服务；需要包含该模块的完整 JDK/JRE。编译、运行：

```text
javac --add-modules jdk.httpserver -encoding UTF-8 HttpLoopbackDemo.java
java --add-modules jdk.httpserver HttpLoopbackDemo
```

```java
import com.sun.net.httpserver.HttpServer;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.logging.Logger;
public class HttpLoopbackDemo {
    private static final Logger logger = Logger.getLogger(HttpLoopbackDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：请求本机 HTTP 服务
         * ========================================================================
         * 数据源：本机临时服务和固定响应
         * 操作：
         * 1) 启动回环服务并构造请求
         * 2) 检查状态码和响应内容
         */
        logger.info("开始 HTTP 本机示例");
        // 1.1 在本机临时端口创建演示服务
        HttpServer server = HttpServer.create(new InetSocketAddress("127.0.0.1", 0), 0);
        server.createContext("/hello", exchange -> {
            /*
             * ========================================================================
             * 步骤1：返回固定示例响应
             * ========================================================================
             * 数据源：本机收到的 HTTP 请求
             * 操作：
             * 1) 编码固定响应
             * 2) 发送状态码和响应正文
             */
            logger.info("开始处理本机请求");
            // 1.1 准备并发送 UTF-8 响应
            byte[] response = "hello".getBytes(StandardCharsets.UTF_8);
            exchange.getResponseHeaders().set("Content-Type", "text/plain; charset=UTF-8");
            exchange.sendResponseHeaders(200, response.length);
            // 1.2 写出响应并关闭本次响应流
            try (OutputStream body = exchange.getResponseBody()) {
                body.write(response);
            }
            logger.info("本机请求处理完成");
        });
        server.start();
        try {
            // 1.2 发起带连接和请求超时的 GET
            HttpClient client = HttpClient.newBuilder()
                    .connectTimeout(Duration.ofSeconds(5)).build();
            URI uri = URI.create("http://127.0.0.1:" + server.getAddress().getPort() + "/hello");
            HttpRequest request = HttpRequest.newBuilder(uri)
                    .timeout(Duration.ofSeconds(5)).GET().build();
            HttpResponse<String> response = client.send(request,
                    HttpResponse.BodyHandlers.ofString(StandardCharsets.UTF_8));
            System.out.println("status=" + response.statusCode());
            System.out.println("body=" + response.body());
        } finally {
            // 1.3 关闭示例服务
            server.stop(0);
        }
        logger.info("HTTP 本机示例完成");
    }
}
```

标准输出：

```text
status=200
body=hello
```

日志写到标准错误。本机服务只绑定 127.0.0.1 的临时端口，程序结束时关闭；此验证不涉及外部网站、TLS、认证或真实业务服务。

## 执行过程

服务端为 `/hello` 返回状态码 200、UTF-8 文本和对应内容长度。客户端创建 HttpClient，设置连接超时；请求本身也设置超时。`send` 同步等待响应，`statusCode()` 与 `body()` 分别读取 HTTP 状态和正文。finally 停止示例服务。

`BodyHandlers.ofString` 会把响应正文收集为字符串；本例响应受控且很小，真实外部响应不能无条件假设大小安全。`send` 可能抛 I/O 或中断异常；本例向外传播便于观察，生产代码需按调用链正确处理并遵守中断约定。

## 工作中怎么用

区分连接超时、请求超时及其他超时，配置以所用客户端语义为准。检查 HTTP 状态、响应类型与业务错误；对外部响应限制大小，避免把错误页面当作成功 JSON。接收包含敏感信息的响应时，不要直接全部写日志。

网络超时不代表服务端没有处理请求。重试前判断方法语义和业务幂等性；涉及扣款、创建订单等写操作，应由双方协议支持幂等键、去重及失败恢复。Java 11 的 HttpClient 与较新 JDK 版本的资源接口存在差异，按项目 JDK 文档处理；使用流式响应时应按契约关闭响应流。

生产 HTTPS 要验证证书链与主机名，不要用“信任所有证书”绕过问题。本例只演示本机明文 HTTP，不能推断网络、代理或 TLS 配置正确。

## 手练与答案

把服务响应改为 `"hi"`：状态仍是 200，body 变为 hi。把请求路径改成未配置的路径：服务可能返回 404，客户端仍可能正常得到 HttpResponse；必须检查状态码。

参考：[廖雪峰：HTTP编程](https://liaoxuefeng.com/books/java/network/http/index.html)；[Oracle HttpClient](https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html)。
