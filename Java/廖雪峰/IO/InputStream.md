# InputStream

> **重要程度：高**｜**手练：需要**
> **重点**：字节读取、EOF 与关闭。

## 核心概念

InputStream 是字节输入抽象。单字节 read 在到达末尾时返回 -1；批量读取应根据实际返回数量处理，不能假定每次都填满缓冲区。

## 工作中注意

用 try-with-resources 管理资源；读取大文件时分块处理，不随意把全部内容放进内存。字节转文本要使用约定编码。

## 手练

分块读取一个测试文件，累计实际字节数，并与文件大小比较。

来源：[教程：InputStream](https://liaoxuefeng.com/books/java/io/input-stream/index.html)。

## 关联知识与实践

`InputStream` / `OutputStream` 处理字节；`Reader` / `Writer` 处理字符。字节转文本要约定编码，例如 UTF-8；图片、压缩包等二进制内容不要随意按文本解码。缓冲流减少频繁的小块读写；实际性能需测量。

来源：[教程：IO](https://liaoxuefeng.com/books/java/io/index.html)；[Java 文件 IO](https://dev.java/learn/java-io/)。
