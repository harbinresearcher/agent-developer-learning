# Maven 与项目构建

> **重要程度：高**｜**手练：需要**
> **重点**：坐标、依赖范围、生命周期、插件、版本冲突与 Maven Wrapper。

## 基本概念

Maven 用 `pom.xml` 描述项目。依赖坐标由 `groupId`、`artifactId`、`version` 等组成；依赖可能继续引入其他传递依赖。依赖范围如 `compile`、`test`、`runtime` 会影响不同阶段的可见性。

常用生命周期阶段依次包括 `validate`、`compile`、`test`、`package`、`verify`、`install` 和 `deploy`；调用后续阶段时会先经过前面的阶段。插件负责具体工作，生命周期阶段本身不是插件。

团队项目优先遵循仓库说明。Maven Wrapper（`mvnw`）可固定项目使用的 Maven 版本；JDK 版本仍需单独满足项目要求。构建失败时先看第一处实际错误，再区分代码错误、依赖下载和环境配置问题。

## 工作中

定期检查依赖树和版本冲突；不要把仓库凭据提交到 `pom.xml`。发布制品是外部写操作，应确认坐标、仓库和发布权限；本地 `install` 不等于发布到远程仓库。

## 手练

创建一个 Maven 项目，添加测试依赖，运行测试与打包；查看依赖树，说明 `test` 范围依赖是否属于生产运行时依赖。

来源：[教程：Maven基础](https://liaoxuefeng.com/books/java/maven/index.html)；[Apache Maven 生命周期](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)。
