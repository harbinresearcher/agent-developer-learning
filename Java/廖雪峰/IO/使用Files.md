# 使用Files

> **重要程度：高**｜**手练：需要**
> **重点**：Path、读写、遍历和资源。

## 核心概念

`Files` 与 `Path` 提供文件和目录操作，例如读取、写入、复制、移动及遍历。具体操作可能抛出 IOException。

## 工作中注意

一次性读取适合规模可控的文件；大文件应流式处理。目录遍历返回的流需要关闭；外部提供路径需验证是否越出允许范围。

## 手练

创建临时目录，写入两个文件并列出，再在结束时清理；检查异常路径下的资源释放。

来源：[教程：使用Files](https://liaoxuefeng.com/books/java/io/files/index.html)。

## 关联知识与实践

使用 `try-with-resources` 确保文件和流关闭。`Path` 表达路径，`Files` 提供读取、写入、遍历等操作；大文件优先流式处理，不要无条件一次读入内存。相对路径相对于进程工作目录，不能假设总是源码目录。

读取 UTF-8 文本并统计行数；复制一个二进制文件，比较复制前后的摘要；测试文件不存在和提前中断时资源是否关闭。

来源：[教程：IO](https://liaoxuefeng.com/books/java/io/index.html)；[Java 文件 IO](https://dev.java/learn/java-io/)。
