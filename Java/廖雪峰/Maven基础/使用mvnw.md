# 使用 Maven Wrapper

> **重要程度：高**｜**手练：需要**
> **重点**：mvnw 脚本、distributionUrl、Maven 与 JDK 版本、首次下载。

## 解决什么问题

不同开发者本机安装的 Maven 版本可能不同。Maven Wrapper 将脚本和目标 Maven 版本配置放在项目中；执行脚本时，它可下载并使用项目指定的 Maven，而不必依赖 PATH 上已有的 Maven。

## 完整练习：为测试项目生成 Wrapper

先按 [[Maven介绍]] 创建 calculator 项目，并确认本机已有 Maven 和 JDK。在该项目根目录运行以下命令，生成示例 Wrapper：

```text
mvn org.apache.maven.plugins:maven-wrapper-plugin:3.3.4:wrapper -Dmaven=3.9.16 -Dtype=only-script
```

生成的主要文件包括 `mvnw`、`mvnw.cmd` 和 `.mvn/wrapper/maven-wrapper.properties`。本次生成的配置为：

```properties
wrapperVersion=3.3.4
distributionType=only-script
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.16/apache-maven-3.9.16-bin.zip
```

Windows PowerShell 中运行：

```text
.\mvnw.cmd -version
.\mvnw.cmd clean verify
```

首次执行可能下载 Maven；之后通常复用缓存。`-version` 应显示 Maven 3.9.16 与实际运行它的 Java 版本；对 calculator 示例，verify 应运行 1 条测试并生成 JAR。本次已在隔离目录生成 only-script Wrapper，并用它运行 `clean verify`，测试 1 条、失败 0 条。

## 按顺序理解

生成 Wrapper 时需要一个已有 Maven 来执行插件；项目已经带 Wrapper 后，其他开发者可直接运行项目脚本。`wrapperVersion` 是 Wrapper 组件版本，`distributionUrl` 指向 Maven 发行包；两者不必相同。only-script 形式与包含 wrapper jar 或源码下载器的形式不同，项目应提交该形式需要的完整文件。

Wrapper 统一 Maven 发行版，但不会自动安装或固定应用所需的 JDK。先看 `mvnw.cmd -version` 中的 Java 信息，再按项目要求检查编译目标和工具链。修改 distributionUrl、校验和或镜像前，应按团队供应链和安全策略核实来源。

## 工作中怎么用

在已有仓库优先使用其自带 Wrapper，不必在笔记练习中覆盖其他项目的 Wrapper。下载失败要区分网络、代理、仓库和权限；不要因为 Maven 脚本成功启动就认定测试已经通过。测试结果需看构建日志和报告。

## 手练与答案

1. 系统 `mvn -version` 与 `mvnw.cmd -version` 显示不同 Maven 版本是可能的：前者使用 PATH，后者使用项目配置的发行版。
2. 将目标 JDK 配置错误时，Wrapper 也不会自动修复；需根据项目要求设置合适 JDK。

参考：[廖雪峰：使用mvnw](https://liaoxuefeng.com/books/java/maven/wrapper/index.html)；[Apache Maven Wrapper](https://maven.apache.org/tools/wrapper/)。
