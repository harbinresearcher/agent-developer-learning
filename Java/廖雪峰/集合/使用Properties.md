# 使用Properties

> **重要程度：中**｜**手练：建议**
> **重点**：配置读取、默认值与凭据。

## 核心概念

Properties 常用于表示字符串形式的配置项，可从文件或流加载；获取缺失项时可指定默认值。

## 工作中注意

不同加载方法对编码的处理不同，应明确 Reader、InputStream 与文件格式的约定。真实密码和令牌不要提交到代码仓库。

## 手练

从测试配置中读取端口；分别验证缺失项、非法数字和编码问题。

来源：[教程：使用Properties](https://liaoxuefeng.com/books/java/collection/properties/index.html)。
