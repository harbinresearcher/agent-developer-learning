# 编写JUnit测试

> **重要程度：高**｜**手练：需要**
> **重点**：测试结构、断言和运行。

## 核心概念

JUnit 测试围绕可观察行为组织：准备输入、调用目标、断言结果。实际注解与运行方式以项目 JUnit 版本和构建配置为准。

## 工作中注意

测试需要能够独立重复运行；测试通过说明已有断言通过，不能证明全部情况正确。

## 手练

为计算折扣的方法分别测试正常输入、边界值和非法输入。

来源：[教程：编写JUnit测试](https://liaoxuefeng.com/books/java/unit-test/junit-test/index.html)。

## 关联知识与实践

测试描述可观察行为：准备输入、调用目标、断言结果。正常值、边界值和失败路径都应考虑。JUnit 提供断言、异常测试、测试前后准备与清理，以及参数化测试；具体注解和版本按项目依赖确定。

单元测试应尽量独立、可重复，不依赖真实网络或时间；需要验证数据库、框架配置和多个组件协作时，再编写相应集成测试。测试通过只证明已覆盖的检查通过。

来源：[教程：单元测试](https://liaoxuefeng.com/books/java/unit-test/index.html)；[Java Pattern](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/Pattern.html)。
