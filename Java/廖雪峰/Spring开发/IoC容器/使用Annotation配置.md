# 使用Annotation配置

> **重要程度：高**｜**手练：需要**
> **重点**：组件扫描与配置注解。

## 核心概念

注解可以声明组件、配置类和依赖注入，容器扫描或加载相关配置后创建对象。

## 工作中注意

注解本身不执行注入；包扫描范围、容器启动方式和框架版本决定是否生效。

## 手练

把测试服务标记为组件，验证位于扫描范围内外时容器是否创建。

来源：[教程：使用Annotation配置](https://liaoxuefeng.com/books/java/spring/ioc/annotation-config/index.html)。
