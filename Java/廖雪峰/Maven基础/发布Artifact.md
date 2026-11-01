# 发布Artifact

> **重要程度：低**｜**手练：按需**
> **重点**：本地安装、远程发布与凭据。

## 核心概念

构建制品可安装到本地仓库，也可发布到配置的远程仓库供其他项目使用。

## 工作中注意

远程发布会改变外部状态，应检查坐标、目标仓库和发布权限；凭据应放在受控配置而非源码。

## 手练

在本地测试仓库安装一个制品并让另一项目引用；说明这与远程发布的区别。

来源：[教程：发布Artifact](https://liaoxuefeng.com/books/java/maven/deploy/index.html)。

## 关联知识与实践

定期检查依赖树和版本冲突；不要把仓库凭据提交到 `pom.xml`。发布制品是外部写操作，应确认坐标、仓库和发布权限；本地 `install` 不等于发布到远程仓库。

来源：[教程：Maven基础](https://liaoxuefeng.com/books/java/maven/index.html)；[Apache Maven 生命周期](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)。
