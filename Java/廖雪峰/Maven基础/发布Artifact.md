# 发布 Artifact

> **重要程度：中**｜**手练：建议**
> **重点**：制品坐标、本地安装、远程部署、仓库与版本边界。

## 先分清三种结果

`package` 在项目构建目录产生制品；`install` 将制品安装到 Maven **本地仓库**；`deploy` 将制品部署到配置的目标仓库，该目标可以是远程服务。三者都不是“发布到生产运行环境”的同义词。

## 本地练习：安装到隔离仓库

先按 [[Maven介绍]] 建立并验证 calculator 项目。在项目目录的 PowerShell 中运行：

```powershell
$notesRepo = Join-Path (Get-Location).Path "notes-m2"
mvn "-Dmaven.repo.local=$notesRepo" install
```

命令会在当前练习目录的 `notes-m2` 中创建本地仓库，并把 `example.notes:calculator:1.0.0` 对应的 POM 和 JAR 安装到坐标路径。查看构建日志及该目录，不要把它误认为远程发布。本地安装会运行到 install 阶段，实际测试是否执行仍需看报告。

本次验证使用 Maven 3.9.16 和单独的本地仓库；一次 `mvn clean verify` 实际运行 1 条测试并通过，另一次 install 验证仅检查了本地制品且显式跳过测试。两次证据不应混为一次完整发布验证。

## 远程仓库是什么

Maven 可通过 `distributionManagement` 或部署参数指定目标仓库，deploy 插件按坐标上传 POM、JAR 等制品和元数据。`settings.xml` 中的服务器 ID 与仓库配置通常需要对应；凭据应放在项目允许的受控配置或密钥管理中，不要写进 pom、源码或日志。

SNAPSHOT 与正式版本的更新、覆盖、保留策略取决于仓库和团队发布流程；不能假设同一正式版本可被安全重复覆盖。发布前要检查构建产物、版本、仓库地址、权限和实际测试结果。实际 deploy 会改变目标仓库，不要为了验证笔记随意向公共仓库上传。

## 仅本地演示部署的边界

本次另用 Maven deploy 插件将测试项目部署到工作区内独立的 `file:` 仓库，且该次实际运行 1 条测试、0 失败；没有上传远程服务。文件仓库示例只能说明制品在本地目标路径的组织方式，不证明远程仓库权限、签名、同步或正式发布流程可用。

## 手练与答案

1. `mvn package` 后能否保证另一个项目按坐标从本地仓库解析到新包？不能；需要安装或通过其他方式提供制品。
2. 本地 install 成功是否意味着远程仓库已收到？不是；它只写入指定本地仓库。
3. 已配置远程 deploy 时能否把命令当只读检查？不能；它可能向仓库上传制品和元数据。

参考：[廖雪峰：发布Artifact](https://liaoxuefeng.com/books/java/maven/deploy/index.html)；[Apache Deploy Plugin](https://maven.apache.org/plugins/maven-deploy-plugin/)。
