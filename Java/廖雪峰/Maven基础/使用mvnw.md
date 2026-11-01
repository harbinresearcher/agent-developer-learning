# 使用mvnw

> **重要程度：中**｜**手练：建议**
> **重点**：Wrapper 和版本一致。

## 核心概念

Maven Wrapper 让项目按配置使用指定 Maven 发行版，减少团队成员本机 Maven 版本差异。

## 工作中注意

Wrapper 不替代 JDK 安装，也不保证网络与远程仓库可用；Windows 与类 Unix 的启动脚本不同。

## 手练

查看项目 Wrapper 配置并运行版本命令，记录 Maven 和 JDK 版本。

来源：[教程：使用mvnw](https://liaoxuefeng.com/books/java/maven/wrapper/index.html)。

## 关联知识与实践

团队项目优先遵循仓库说明。Maven Wrapper（`mvnw`）可固定项目使用的 Maven 版本；JDK 版本仍需单独满足项目要求。构建失败时先看第一处实际错误，再区分代码错误、依赖下载和环境配置问题。

来源：[教程：Maven基础](https://liaoxuefeng.com/books/java/maven/index.html)；[Apache Maven 生命周期](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)。
