# OutputStream

> **重要程度：高**｜**手练：需要**
> **重点**：字节写入、flush 和关闭。

## 核心概念

OutputStream 提供字节输出；可以单字节或写入字节数组的一部分。缓冲流可能暂存数据，flush 请求将缓冲内容推向下层流。

## 工作中注意

关闭资源与持久化到物理介质不是完全相同的保证；不要把 flush 当成数据库提交或断电持久化保证。

## 手练

向临时文件写入 UTF-8 字节，关闭后读取并核对原内容。

来源：[教程：OutputStream](https://liaoxuefeng.com/books/java/io/output-stream/index.html)。
