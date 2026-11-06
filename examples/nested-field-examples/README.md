# StreamSQL 嵌套字段访问功能完整演示

这个演示展示了 StreamSQL 的完整嵌套字段访问功能，包括基础点号访问、复杂数组索引、Map键访问和混合操作。

## 功能概览

### 基础功能
- **点号语法访问**：使用 `field.subfield.property` 访问嵌套结构
- **条件过滤**：在WHERE子句中使用嵌套字段进行数据筛选
- **聚合计算**：对嵌套字段进行GROUP BY和聚合函数操作

### 高级功能
- **数组索引访问**：使用 `array[0]`、`array[-1]` 等语法访问数组元素
- **Map键访问**：支持 `map['key']` 和 `map["key"]` 语法访问Map值
- **混合复杂访问**：组合使用点号、数组索引、Map键进行深层嵌套访问
- **负数索引**：支持负数索引从数组末尾开始访问元素
- **复杂聚合**：在聚合函数中使用复杂字段访问路径

## 运行演示

```bash
cd examples/nested-field-examples
go run main.go
```

## 演示内容

### 第一部分：基础嵌套字段访问

#### 示例SQL
```sql
SELECT device.info.name as device_name, 
       device.location,
       sensor.temperature,
       sensor.humidity
FROM stream 
WHERE device.location = 'room-A' 
  AND sensor.temperature > 20
```

#### 测试数据结构
```json
{
  "device": {
    "info": {
      "name": "温度传感器-001",
      "type": "temperature"
    },
    "location": "room-A"
  },
  "sensor": {
    "temperature": 25.5,
    "humidity": 60.2
  }
}
```

### 第二部分：嵌套字段聚合

#### 示例SQL
```sql
SELECT device.location, 
       AVG(sensor.temperature) as avg_temp,
       MAX(sensor.humidity) as max_humidity,
       COUNT(*) as sensor_count
FROM stream 
GROUP BY device.location, TumblingWindow('2s')
WITH (TIMESTAMP='timestamp', TIMEUNIT='ss')
```

### 第三部分：复杂嵌套字段访问

#### 演示1：数组索引访问

**SQL查询**
```sql
SELECT device, 
       sensors[0].temperature as first_sensor_temp,
       sensors[1].humidity as second_sensor_humidity,
       data[2] as third_data_item
FROM stream
```

**数据结构**
```json
{
  "device": "工业传感器-001",
  "sensors": [
    {"temperature": 25.5, "humidity": 60.2},
    {"temperature": 26.8, "humidity": 58.7},
    {"temperature": 24.1, "humidity": 62.1}
  ],
  "data": ["status_ok", "battery_95%", "signal_strong", "location_A1"]
}
```

#### 演示2：Map键访问

**SQL查询**
```sql
SELECT device_id,
       config['host'] as server_host,
       config["port"] as server_port,
       settings['enable_ssl'] as ssl_enabled,
       metadata["version"] as app_version
FROM stream
```

**数据结构**
```json
{
  "device_id": "gateway-001",
  "config": {
    "host": "192.168.1.100",
    "port": 8080,
    "protocol": "https"
  },
  "settings": {
    "enable_ssl": true,
    "timeout": 30
  }
}
```

#### 演示3：混合复杂访问

**SQL查询**
```sql
SELECT building,
       floors[0].rooms[2]['name'] as first_floor_room3_name,
       floors[1].sensors[0].readings['temperature'] as second_floor_first_sensor_temp,
       metadata.building_info['architect'] as building_architect,
       alerts[-1].message as latest_alert
FROM stream
```

**数据结构**
```json
{
  "building": "智能大厦A座",
  "floors": [
    {
      "floor_number": 1,
      "rooms": [
        {"name": "大厅", "type": "public"},
        {"name": "接待室", "type": "office"},
        {"name": "会议室A", "type": "meeting"}
      ]
    },
    {
      "floor_number": 2,
      "sensors": [
        {
          "id": "sensor-201",
          "readings": {
            "temperature": 23.5,
            "humidity": 58.2
          }
        }
      ]
    }
  ]
}
```

#### 演示4：负数索引访问

**SQL查询**
```sql
SELECT device_name,
       readings[-1] as latest_reading,
       history[-2] as second_last_event,
       tags[-1] as last_tag
FROM stream
```

**数据结构**
```json
{
  "device_name": "温度监测器-Alpha",
  "readings": [18.5, 19.2, 20.1, 21.3, 22.8, 23.5],
  "history": ["boot", "calibration", "running", "alert", "resolved"],
  "tags": ["indoor", "critical", "monitored"]
}
```

#### 演示5：数组索引聚合计算

**SQL查询**
```sql
SELECT location,
       AVG(sensors[0].temperature) as avg_first_sensor_temp,
       MAX(sensors[1].humidity) as max_second_sensor_humidity,
       COUNT(*) as device_count
FROM stream 
GROUP BY location, TumblingWindow('2s')
WITH (TIMESTAMP='timestamp', TIMEUNIT='ss')
```

## 支持的访问语法总结

| 语法类型 | 示例 | 说明 |
|---------|------|------|
| 点号访问 | `device.info.name` | 基础嵌套字段访问 |
| 数组正索引 | `items[0]`, `data[2]` | 从0开始的数组索引 |
| 数组负索引 | `items[-1]`, `data[-2]` | 从末尾开始的数组索引 |
| Map单引号键 | `config['host']` | 使用单引号的Map键访问 |
| Map双引号键 | `settings["timeout"]` | 使用双引号的Map键访问 |
| 混合访问 | `users[0].profile['name']` | 组合多种访问方式 |
| 多维数组 | `matrix[1][2]` | 二维或多维数组访问 |

## 预期输出示例

### 基础嵌套字段访问结果
```
📊 第一部分：基础嵌套字段访问
  📋 基础嵌套字段访问结果:
    记录 1:
      设备名称: 温度传感器-001
      设备位置: room-A
      温度: 25.5°C
      湿度: 60.2%
```

### 数组索引访问结果
```
📊 演示1: 数组索引访问
  📋 数组索引访问结果:
    记录 1:
      设备: 工业传感器-001
      第一个传感器温度: 25.5°C
      第二个传感器湿度: 58.7%
      第三个数据项: signal_strong
```

### Map键访问结果
```
🗝️ 演示2: Map键访问
  🗝️ Map键访问结果:
    记录 1:
      设备ID: gateway-001
      服务器主机: 192.168.1.100
      服务器端口: 8080
      SSL启用: true
      应用版本: v2.1.3
```

### 聚合计算结果
```
📈 演示5: 数组索引聚合计算
  📈 数组索引聚合计算结果:
    聚合结果 1:
      位置: 车间A
      第一个传感器平均温度: 24.83°C
      第二个传感器最大湿度: 68.2%
      设备数量: 4
```

## 技术特性

### 性能优化
- 字段路径解析结果缓存
- 高效的嵌套访问算法
- 最小化内存分配

### 错误处理
- 安全的数组边界检查
- 优雅的空值处理
- 类型安全的访问操作

### 兼容性
- 完全向后兼容简单字段访问
- 支持所有现有SQL功能
- 无缝集成聚合和窗口函数

## 使用建议

1. **字段别名**：为复杂路径使用AS别名提高可读性
2. **数据验证**：在生产环境中验证数据结构的一致性
3. **性能考虑**：避免过深的嵌套层级（建议不超过5层）
4. **错误预期**：预期并处理字段不存在的情况
5. **测试覆盖**：全面测试各种数据结构和边界情况

## 注意事项

- 数组索引从0开始，负数索引从-1开始
- Map键访问区分大小写
- 访问不存在的字段返回null而不抛出异常
- 所有操作都是线程安全的
- 支持实时流处理和批处理模式 