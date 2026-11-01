# UDP编程

> **重要程度：高**｜**手练：需要**
> **重点**：数据报边界、长度、超时和不可靠性。

## 解决什么问题

UDP 传送彼此独立的数据报，不提供 TCP 那样的连接内可靠有序字节流。它适用于协议能够接受或自行处理丢包、重复和乱序的需求；是否合适要看业务，而不是简单地认为 UDP 一定更快。

## 完整例子

保存为 `UdpLoopbackDemo.java`，编译运行；日志写到标准错误。示例仅向本机临时端口发送一个短数据报。

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.charset.StandardCharsets;
import java.util.logging.Logger;
public class UdpLoopbackDemo {
    private static final Logger logger = Logger.getLogger(UdpLoopbackDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：在本机发送和接收数据报
         * ========================================================================
         * 数据源：本机临时端口和固定短消息
         * 操作：
         * 1) 发送一个 UDP 数据报
         * 2) 按实际接收长度解码
         */
        logger.info("开始 UDP 本机示例");
        // 1.1 创建本机接收端和发送端
        InetAddress loopback = InetAddress.getByName("127.0.0.1");
        try (DatagramSocket receiver = new DatagramSocket(0, loopback);
             DatagramSocket sender = new DatagramSocket()) {
            receiver.setSoTimeout(5000);
            byte[] data = "hello".getBytes(StandardCharsets.UTF_8);
            sender.send(new DatagramPacket(data, data.length, loopback, receiver.getLocalPort()));
            // 1.2 接收一个报文并只读取有效字节
            byte[] buffer = new byte[64];
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
            receiver.receive(packet);
            String text = new String(packet.getData(), packet.getOffset(), packet.getLength(), StandardCharsets.UTF_8);
            System.out.println("received=" + text);
            System.out.println("bytes=" + packet.getLength());
        }
        logger.info("UDP 本机示例完成");
    }
}
```

标准输出：

```text
received=hello
bytes=5
```

## 执行过程

接收端绑定本机回环地址和系统分配的端口；发送端把 UTF-8 编码的 hello 放入一个 DatagramPacket 并发送。接收端最多等待 5 秒，随后按 packet 的 offset 和实际 length 解码，得到 hello 和 5 个字节。

接收缓冲区太小时，数据报可能被截断；不能把整个预分配 buffer 当成有效内容。发送成功不等于对方已经接收。示例只验证本机一次收发，不证明跨机器网络可靠，也不提供重试或身份认证。

## 工作中怎么用

协议需要明确数据报大小、编码、超时、重复处理、消息顺序及必要的完整性和认证机制。网络可能丢包、重排或重复，不能依赖到达顺序；大数据报还可能带来分片与丢失问题。日志应避免包含敏感报文。

如果业务需要可靠交付，应选择合适的成熟协议，或在明确边界下设计确认、序号和重试；重试写操作同样要考虑幂等。本例不包含这些能力。

## 手练与答案

消息换成 `"Hi"`：接收到文本 Hi，UTF-8 字节数为 2。若没有数据报到达，`receive` 在本例设置的超时后抛异常，不会自动收到空消息。

参考：[廖雪峰：UDP编程](https://liaoxuefeng.com/books/java/network/udp/index.html)；[Oracle 文档](https://docs.oracle.com/javase/tutorial/networking/datagrams/definition.html)。
