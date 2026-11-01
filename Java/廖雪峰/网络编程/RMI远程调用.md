# RMI 远程调用

> **重要程度：低**｜**手练：按需**
> **重点**：远程接口、stub、网络失败、序列化与安全边界。

## 解决什么问题

Java RMI 允许通过远程接口调用另一个 Java 虚拟机中的对象。调用看起来像普通方法，但实际可能涉及网络、参数传输、序列化和远程异常。它常见于旧项目；新系统的通信方案应按项目约定和维护需要选择。

## 完整例子：只在本机调用远程引用

保存为 `RmiLoopbackDemo.java`，编译并运行。程序在当前 JVM 中导出一个只监听 127.0.0.1 的临时远程对象，再通过 RMI stub 调用，最后取消导出；日志写到标准错误。它不访问外部服务，也不使用 RMI Registry。

```java
import java.net.InetAddress;
import java.net.ServerSocket;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.logging.Logger;
public class RmiLoopbackDemo {
    private static final Logger logger = Logger.getLogger(RmiLoopbackDemo.class.getName());
    public static void main(String[] args) throws Exception {
        /*
         * ========================================================================
         * 步骤1：导出并调用本地远程对象
         * ========================================================================
         * 数据源：仅限本机的演示服务
         * 操作：
         * 1) 导出实现并获取远程引用
         * 2) 调用方法并取消导出
         */
        logger.info("开始 RMI 本机示例");
        // 1.1 限制服务监听和引用地址为 IPv4 回环
        System.setProperty("java.rmi.server.hostname", "127.0.0.1");
        GreetingImpl target = new GreetingImpl();
        Greeting stub = (Greeting) UnicastRemoteObject.exportObject(target, 0, null,
                port -> new ServerSocket(port, 0, InetAddress.getByName("127.0.0.1")));
        try {
            // 1.2 通过远程引用调用接口方法
            System.out.println("result=" + stub.greet("Ada"));
        } finally {
            // 1.3 释放演示用远程对象
            UnicastRemoteObject.unexportObject(target, true);
        }
        logger.info("RMI 本机示例完成");
    }
    public interface Greeting extends Remote {
        String greet(String name) throws RemoteException;
    }
    public static class GreetingImpl implements Greeting {
        public String greet(String name) throws RemoteException {
            /*
             * ========================================================================
             * 步骤1：生成问候文本
             * ========================================================================
             * 数据源：远程调用传入的姓名
             * 操作：
             * 1) 组合响应
             * 2) 返回响应内容
             */
            logger.info("开始生成问候");
            // 1.1 生成示例响应
            String result = "Hello, " + name;
            logger.info("问候生成完成");
            return result;
        }
    }
}
```

标准输出为 `result=Hello, Ada`。本例已用 Java 11 编译运行。`java.rmi.server.hostname` 是当前 JVM 范围的设置，本例仅在独立演示进程中设为回环地址；不要在已有服务中不加评估地修改全局网络配置。

## 执行过程

`Greeting` 继承 `Remote`，远程方法声明 `RemoteException`。`GreetingImpl` 提供实现。`UnicastRemoteObject.exportObject` 导出对象并返回 stub；示例自定义服务端 socket 工厂，只监听本机地址，端口 0 表示由系统选择端口。调用 `stub.greet("Ada")` 经过 RMI 调用，得到响应；finally 取消导出，释放示例服务。

这个示例直接持有导出得到的 stub，没有演示跨进程服务发现。跨进程应用可以使用 RMI Registry 绑定名称并查找；`LocateRegistry.getRegistry` 取得引用不等于已经成功连接，实际调用注册表时仍可能失败。部署时还要处理网络可达性、服务端地址和资源生命周期。

## 工作中怎么用

远程调用可能超时、断连或在响应丢失前已经执行；不能当作零成本、可靠的本地调用，也不能盲目重试有副作用的操作。设计超时、幂等、授权和可观测性，并处理 `RemoteException` 与业务异常。

RMI 可能涉及对象反序列化；暴露给不可信网络存在安全风险。必须按项目安全方案限制网络、身份和可反序列化类型，不能把示例当作生产安全配置。新项目通常应评估团队支持的标准协议，而不是仅因 RMI 语法简单而选用。

## 手练与答案

把姓名改成 `"Bo"`，输出 `result=Hello, Bo`。把对 `stub.greet` 的调用改为直接调用 `target.greet`：返回值可能相同，但后者是本地普通调用，不再验证 RMI 调用链。

参考：[廖雪峰：RMI远程调用](https://liaoxuefeng.com/books/java/network/rmi/index.html)；[Oracle RMI Registry](https://docs.oracle.com/en/java/javase/17/docs/specs/rmi/registry.html)。
