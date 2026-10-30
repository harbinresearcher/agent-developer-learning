# 使用EnumMap

> **重要程度：中**｜**手练：建议**
> **重点**：枚举键和紧凑映射。

## 核心概念

EnumMap 专门使用同一种枚举类型作为键，适合按有限状态保存关联值；其键的迭代顺序按枚举声明顺序。

## 工作中注意

枚举键类型需要明确；不要假设它支持 null 键。是否允许 null 值应查具体 API。对外持久化仍应使用稳定状态码而非依赖 ordinal。

## 手练

用枚举表示订单状态，统计每个状态的订单数量。

来源：[教程：使用EnumMap](https://liaoxuefeng.com/books/java/collection/enum-map/index.html)。
