# 使用Resource

> **重要程度：中**｜**手练：建议**
> **重点**：资源抽象与路径。

## 核心概念

Spring Resource 统一表达类路径、文件等来源的资源，并提供访问方式。

## 工作中注意

classpath 资源打包后不一定是普通文件；读取时优先使用适合的流并关闭。

## 手练

把文本放入资源目录，在打包前后读取并核对内容。

来源：[教程：使用Resource](https://liaoxuefeng.com/books/java/spring/ioc/resource/index.html)。
