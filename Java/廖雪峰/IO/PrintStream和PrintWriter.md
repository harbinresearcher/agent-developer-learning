# PrintStream和PrintWriter

> **重要程度：中**｜**手练：按需**
> **重点**：格式化输出和错误检查。

## 核心概念

PrintStream 和 PrintWriter 提供 print、println、printf 等便捷输出；前者面向字节输出，后者面向字符输出。

## 工作中注意

这些类的错误处理方式与直接抛 IOException 的流不同，必要时使用 API 提供的错误检查方法；明确编码和关闭职责。

## 手练

把格式化报表写到测试文件，并验证编码、换行和输出内容。

来源：[教程：PrintStream和PrintWriter](https://liaoxuefeng.com/books/java/io/print/index.html)。
