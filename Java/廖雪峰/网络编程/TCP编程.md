# TCP编程

> **重要程度：高**｜**手练：需要**
> **重点**：连接、消息分帧、超时、关闭与失败。

## 解决什么问题

TCP 连接上的两端通过字节流通信，保证连接内的字节顺序；它不保留应用消息边界。协议必须规定如何分帧，例如长度前缀或分隔符，并处理连接失败、超时和资源关闭。

## 完整例子

保存为 `TcpLoopbackDemo.java`，编译运行；日志写到标准错误。示例只绑定本机 127.0.0.1，由系统选择临时端口，处理一个连接后关闭。

```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.logging.Logger;
public class TcpLoopbackDemo {
    private static final Logger logger = Logger.getLogger(TcpLoopbackDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：在本机交换一条消息
         * ========================================================================
         * 数据源：本机回环服务与固定短文本
         * 操作：
         * 1) 启动一次性服务并接收消息
         * 2) 客户端读取响应并关闭资源
         */
        logger.info("开始 TCP 本机示例");
        // 1.1 创建只处理一个连接的本机服务
        ExecutorService executor = Executors.newSingleThreadExecutor();
        try (ServerSocket server = new ServerSocket(0, 1, InetAddress.getByName("127.0.0.1"))) {
            server.setSoTimeout(5000);
            Future<String> received = executor.submit(() -> {
                /*
                 * ========================================================================
                 * 步骤1：接收并响应消息
                 * ========================================================================
                 * 数据源：本机客户端连接
                 * 操作：
                 * 1) 按约定格式读取文本
                 * 2) 写回确认消息
                 */
                logger.info("开始接收本机消息");
                // 1.1 读取一条带长度的示例消息
                try (Socket peer = server.accept();
                     DataInputStream in = new DataInputStream(peer.getInputStream());
                     DataOutputStream out = new DataOutputStream(peer.getOutputStream())) {
                    peer.setSoTimeout(5000);
                    String message = in.readUTF();
                    // 1.2 写入响应并刷新
                    out.writeUTF("ACK:" + message);
                    out.flush();
                    logger.info("本机消息处理完成");
                    return message;
                }
            });
            // 1.2 使用短超时连接本机服务并读取响应
            try (Socket client = new Socket()) {
                client.connect(new InetSocketAddress("127.0.0.1", server.getLocalPort()), 5000);
                client.setSoTimeout(5000);
                DataOutputStream out = new DataOutputStream(client.getOutputStream());
                DataInputStream in = new DataInputStream(client.getInputStream());
                out.writeUTF("hello");
                out.flush();
                System.out.println("reply=" + in.readUTF());
            }
            System.out.println("received=" + received.get(5, TimeUnit.SECONDS));
        } finally {
            executor.shutdownNow();
        }
        logger.info("TCP 本机示例完成");
    }
}
```

标准输出：

```text
reply=ACK:hello
received=hello
```

## 执行过程

服务端创建 ServerSocket 并在后台等待本机连接。客户端连接到实际分配的端口，用 `writeUTF` 发送 hello；服务端 `readUTF` 按相同的 Java 格式读取，再写回 ACK:hello。客户端先输出响应，主线程等待后台任务返回再输出收到的消息，避免依赖线程日志顺序。

`writeUTF/readUTF` 使用 Java 定义的长度前缀和 modified UTF-8 格式，适合此教学示例，不等于任意语言或协议约定的普通 UTF-8 帧。跨语言通信应明确长度字节序、编码、最大帧大小和异常处理。

## 工作中怎么用

一次 `read` 不保证得到完整应用消息，需要循环读取并遵守协议边界；`available()` 也不是完整消息长度。设置连接、读取和业务超时，限制输入大小，关闭 socket 和线程池；timeout 不代表对方没有执行操作。

本例没有 TLS、身份认证、多连接并发和可靠重试，不能直接用于生产或公开网络。TLS 需要正确验证证书和主机名；重复写请求还需幂等设计。

## 手练与答案

把客户端消息改成 `"Ada"`：响应为 `ACK:Ada`，收到的是 Ada。服务端关闭或未在超时时间内响应时，客户端可能抛连接或超时异常；不能当作业务成功。

参考：[廖雪峰：TCP编程](https://liaoxuefeng.com/books/java/network/tcp/index.html)；[Oracle 文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/net/Socket.html)。
