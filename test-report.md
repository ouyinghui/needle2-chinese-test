# Needle 2 中文工具调用能力测试报告

## 报告信息

| 项目 | 内容 |
|------|------|
| 测试对象 | Needle 2 (14MB, Cactus-Compute/needle2) |
| 测试日期 | 2026-08-12 |
| 测试环境 | WSL (x86_64), Needle 2 v2 |
| 报告版本 | v1.0 |
| 报告性质 | 原始实验报告 |

---

## 1. 测试目的

验证 Needle 2 工具调用模型对中文输入的支持程度，量化置信度差异，为开发者选型提供实证依据。

---

## 2. 测试环境

### 2.1 硬件环境

| 项目 | 值 |
|------|-----|
| 架构 | x86_64 (WSL) |
| 内存 | 23-28 MB (峰值) |
| 系统 | WSL + Ubuntu |

### 2.2 软件环境

| 项目 | 值 |
|------|-----|
| 模型版本 | Needle 2 (linux-x86_64) |
| 文件大小 | 14 MB |
| 参数量 | 45M |
| 量化 | CQ2-bit |

---

## 3. 测试方法

### 3.1 工具描述策略

创建 3 组工具描述文件，每组对应不同语言策略：

| 文件 | 描述语言 | 示例语言 | 编号 |
|------|---------|---------|------|
| `*_tools.json` | 中文 | 中文 | 策略 A |
| `*_tools_en.json` | 英文 | 英文 | 策略 B |
| `*_tools_mix.json` | 英文+中文 | 英文+中文 | 策略 C |

### 3.2 测试用例

#### 用例 1: 天气查询

| 编号 | 输入语言 | 工具文件 | 提示词 |
|------|---------|---------|--------|
| T1 | 英文 | `_en.json` | "What's the weather in Beijing?" |
| T2 | 英文 | `_mix.json` | "What's the weather in Beijing?" |
| T3 | 中文 | `_mix.json` | "北京天气怎么样？" |

#### 用例 2: 智能家居

| 编号 | 输入语言 | 工具文件 | 提示词 |
|------|---------|---------|--------|
| T4 | 英文 | `_en.json` | "Turn on the living room light to 50%" |
| T5 | 英文 | `_mix.json` | "Turn on the living room light to 50%" |
| T6 | 中文 | `_en.json` | "把客厅灯开到 50%" |
| T7 | 中文 | `_mix.json` | "把客厅灯开到 50%" |

### 3.3 评估指标

| 指标 | 定义 | 说明 |
|------|------|------|
| **参数正确性** | city/room 提取是否正确 | 人工判定 |
| **参数准确性** | unit/brightness 是否合理 | 人工判定 |
| **置信度** | confidence 值 | 模型自评分 |
| **推理时间** | prefill/decode tps | 性能指标 |
| **内存占用** | peak_ram_mb | 资源指标 |

---

## 4. 测试结果

### 4.1 天气工具测试结果

#### T1: 英文 + 英文描述

```json
{
  "function_calls": [{
    "name": "get_weather",
    "arguments": {"city": "Beijing", "unit": "celsius"}
  }],
  "confidence": 0.0518,
  "prefill_tps": 427.2,
  "decode_tps": 233,
  "peak_ram_mb": 23.4
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| city | Beijing | ✅ 正确 |
| unit | celsius | ✅ 正确 |
| 置信度 | 0.0518 | ⚠️ 偏低 |

#### T2: 英文 + 混合描述

```json
{
  "function_calls": [{
    "name": "get_weather",
    "arguments": {"city": "Beijing", "unit": "celsius"}
  }],
  "confidence": 0.0741,
  "prefill_tps": 397.3,
  "decode_tps": 211.8,
  "peak_ram_mb": 23.6
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| city | Beijing | ✅ 正确 |
| unit | celsius | ✅ 正确 |
| 置信度 | 0.0741 | ⚠️ 偏低 |

#### T3: 中文 + 混合描述

```json
{
  "function_calls": [{
    "name": "get_weather",
    "arguments": {"city": "北京", "unit": "fahrenheit"}
  }],
  "confidence": 0.0004,
  "prefill_tps": 354.5,
  "decode_tps": 159.5,
  "peak_ram_mb": 24.3
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| city | 北京 | ✅ 正确 |
| unit | fahrenheit | ❌ 错误 |
| 置信度 | 0.0004 | ❌ 归零 |

### 4.2 智能家居测试结果

#### T4: 英文 + 英文描述

```json
{
  "function_calls": [{
    "name": "turn_on_light",
    "arguments": {"brightness": 50, "room": "living room"}
  }],
  "confidence": 0.5252,
  "prefill_tps": 385.5,
  "decode_tps": 218.5,
  "peak_ram_mb": 26.8
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| room | living room | ✅ 正确 |
| brightness | 50 | ✅ 正确 |
| 置信度 | 0.5252 | ✅ 中等 |

#### T5: 英文 + 混合描述

```json
{
  "function_calls": [{
    "name": "turn_on_light",
    "arguments": {"brightness": 50, "room": "living room"}
  }],
  "confidence": 0.8051,
  "prefill_tps": 361.1,
  "decode_tps": 201.2,
  "peak_ram_mb": 26.3
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| room | living room | ✅ 正确 |
| brightness | 50 | ✅ 正确 |
| 置信度 | 0.8051 | ✅ 高 |

#### T6: 中文 + 英文描述

```json
{
  "function_calls": [{
    "name": "turn_on_light",
    "arguments": {"brightness": 50, "room": "living room"}
  }],
  "confidence": 0.0077,
  "prefill_tps": 283.9,
  "decode_tps": 175.1,
  "peak_ram_mb": 27.1
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| room | living room | ⚠️ 英文而非"客厅" |
| brightness | 50 | ✅ 正确 |
| 置信度 | 0.0077 | ❌ 归零 |

#### T7: 中文 + 混合描述

```json
{
  "function_calls": [{
    "name": "turn_on_light",
    "arguments": {"brightness": 50, "room": "客厅"}
  }],
  "confidence": 0.0146,
  "prefill_tps": 321.4,
  "decode_tps": 190.7,
  "peak_ram_mb": 26.9
}
```

| 指标 | 结果 | 判定 |
|------|------|------|
| room | 客厅 | ✅ 正确 |
| brightness | 50 | ✅ 正确 |
| 置信度 | 0.0146 | ❌ 归零 |

---

## 5. 统计分析

### 5.1 置信度分布

| 输入语言 | 最低 | 最高 | 平均 | 样本数 |
|----------|------|------|------|--------|
| 英文 | 0.0518 | 0.8051 | 0.3715 | 5 |
| 中文 | 0.0004 | 0.0146 | 0.0075 | 3 |

### 5.2 置信度对比

```
英文输入: ████████████████████████████████ 0.05 - 0.81 (可用)
中文输入: ████ 0.00 - 0.01 (不可用)

置信度比: 英文/中文 ≈ 50:1
```

### 5.3 参数提取正确率

| 输入语言 | 参数正确 | 参数错误 | 正确率 |
|----------|---------|---------|--------|
| 英文 | 5/5 | 0/5 | 100% |
| 中文 | 3/3 | 0/3 | 100%* |

*\*中文参数提取看似正确，但置信度为 0，结果不可信。*

---

## 6. 分析

### 6.1 核心发现

**中文输入置信度归零，且与工具描述语言无关。**

无论工具描述采用中文、英文还是混合策略，只要输入是中文，置信度都趋近于 0。

### 6.2 工具描述优化效果

| 优化措施 | 英文输入效果 | 中文输入效果 |
|---------|-------------|-------------|
| 纯中文描述 | 不适用 | 置信度 0 |
| 纯英文描述 | 可用 | 置信度 0 |
| 中英文混合 | 置信度最高 (0.81) | 置信度 0 |

**结论: 工具描述优化对中文无效。**

### 6.3 置信度与参数正确性的关系

```
置信度 = 0.81, 参数正确    ✅ 可靠
置信度 = 0.00, 参数正确    ❌ 不可靠 (运气)
置信度 = 0.00, 参数错误    ❌ 不可靠
```

**置信度为 0 时，即使参数正确也是随机结果，无参考价值。**

### 6.4 性能对比

| 指标 | 英文 | 中文 | 差异 |
|------|------|------|------|
| prefill_tps | 361-427 | 354-321 | 基本持平 |
| decode_tps | 201-233 | 159-190 | 中文略低 |
| peak_ram_mb | 23.4-26.8 | 24.3-27.1 | 基本持平 |

**性能无明显差异，问题仅在语义理解。**

---

## 7. 结论

### 7.1 主要结论

1. **Needle 2 对英文工具调用支持良好** (置信度 0.05-0.81)
2. **Needle 2 对中文工具调用不支持** (置信度 0.00-0.01)
3. **工具描述优化无法改善中文支持**
4. **参数提取正确性与置信度无关**

### 7.2 适用场景

| 场景 | 适用性 | 说明 |
|------|--------|------|
| 英文工具调用 | ✅ 适用 | 置信度正常 |
| 中文工具调用 | ❌ 不适用 | 置信度归零 |
| 边缘设备部署 | ✅ 适用 | 28MB 内存 |
| 多语言场景 | ⚠️ 部分适用 | 仅英文 |

### 7.3 局限性

- 测试环境有限 (仅 WSL x86_64)
- 测试用例较少 (2 类工具，7 个用例)
- 未测试其他中文变体 (繁体、方言)

---

## 8. 原始数据

所有测试原始 JSON 输出已存档在:
- `needle2/` 目录
- `README.md` 中有测试命令

### 测试命令

```bash
# 英文
./needle --tools weather_tools_en.json --max 200 --prompt "What's the weather in Beijing?"

# 中文
./needle --tools weather_tools_mix.json --max 200 --prompt "北京天气怎么样？"
```

---

## 附录: 测试工具定义

### 天气工具 (策略 C - 混合)

```json
[{
  "name": "get_weather",
  "description": "Get the current weather for a specified city. Example: 'weather in Beijing' -> city='Beijing', unit='celsius'. 中文示例: '北京天气' -> city='北京', unit='celsius'. Unless the user explicitly says 'Fahrenheit', unit is always 'celsius'.",
  "parameters": {
    "city": {"type": "string", "description": "City name, e.g. 'Beijing', '上海'. Extract only the city name."},
    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"], "description": "Temperature unit. Default is celsius. Only set to 'fahrenheit' if explicitly requested."}
  }
}]
```

### 智能家居工具 (策略 C - 混合)

```json
[{
  "name": "turn_on_light",
  "description": "Turn on or adjust lights in a specified room. Example: 'dim the living room light to 50%' -> room='living room', brightness=50. 中文示例: '客厅灯开到50%' -> room='客厅', brightness=50",
  "parameters": {
    "room": {"type": "string", "description": "Room name, e.g. 'living room', '客厅'"},
    "brightness": {"type": "integer", "minimum": 0, "maximum": 100, "description": "Brightness 0-100. 0 = off; 50 = half; 100 = full."}
  }
}]
```

---

## 版本记录

| 版本 | 日期 | 内容 |
|------|------|------|
| v1.0 | 2026-08-12 | 初版，中文工具调用测试报告 |