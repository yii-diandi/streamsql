# 自定义函数完整演示

## 简介

这是StreamSQL自定义函数系统的完整功能演示，涵盖了所有函数类型和高级用法。

## 功能演示

- 🔢 **数学函数**：距离计算、温度转换、圆面积计算
- 📝 **字符串函数**：JSON提取、字符串反转、字符串重复
- 🔄 **转换函数**：IP地址转换、字节大小格式化
- 📅 **时间日期函数**：时间格式化、时间差计算
- 📊 **聚合函数**：几何平均数、众数计算
- 🔍 **分析函数**：移动平均值
- 🛠️ **函数管理**：注册、查询、分类、注销

## 运行方式

```bash
cd examples/custom-functions-demo
go run main.go
```

## 代码亮点

### 1. 完整函数类型覆盖
```go
// 数学函数：距离计算
functions.RegisterCustomFunction("distance", functions.TypeMath, ...)

// 字符串函数：JSON提取  
functions.RegisterCustomFunction("json_extract", functions.TypeString, ...)

// 转换函数：IP转换
functions.RegisterCustomFunction("ip_to_int", functions.TypeConversion, ...)
```

### 2. 自定义聚合函数
```go
type GeometricMeanFunction struct {
    *functions.BaseFunction
}

// 配合聚合器使用
aggregator.Register("geometric_mean", func() aggregator.AggregatorFunction {
    return &GeometricMeanAggregator{}
})
```

### 3. 复杂SQL查询
```sql
SELECT 
    device,
    AVG(distance(x1, y1, x2, y2)) as avg_distance,
    json_extract(metadata, 'version') as version,
    format_bytes(memory_usage) as formatted_memory
FROM stream 
GROUP BY device, TumblingWindow('1s')
```

## 演示流程

1. **函数注册阶段** - 注册各类型函数
2. **SQL测试阶段** - 在不同模式下测试函数
3. **管理功能演示** - 展示函数发现和管理功能

## 适用场景

- 🏢 **企业级应用**：了解完整功能特性
- 🔬 **功能验证**：测试复杂函数组合
- �� **学习参考**：最佳实践和使用模式 