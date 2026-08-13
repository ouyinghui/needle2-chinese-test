# Needle 2 中文工具调用测试集

## 核心发现

**中文输入置信度归零 (0.00)**

- ✅ 英文工具调用: 置信度 0.05-0.81
- ❌ 中文工具调用: 置信度 0.00

---

## 测试方法

### 工具描述策略

对比 3 组工具描述文件:

| 文件 | 描述语言 | 示例语言 | 说明 |
|------|---------|---------|------|
| `*_tools.json` | 中文 | 中文 | 纯中文描述 |
| `*_tools_en.json` | 英文 | 英文 | 纯英文描述 |
| `*_tools_mix.json` | 英文+中文 | 英文+中文 | 混合描述 |

### 测试命令

```bash
# 英文输入
./needle --tools weather_tools_en.json --max 200 --prompt "What's the weather in Beijing?"

# 中文输入
./needle --tools weather_tools.json --max 200 --prompt "北京天气怎么样？"

# 混合描述 + 中文输入
./needle --tools weather_tools_mix.json --max 200 --prompt "北京天气怎么样？"
```

---

## 测试结果

### 天气工具

| 输入语言 | 工具文件 | city | unit | 置信度 |
|----------|---------|------|------|--------|
| 英文 | `_en.json` | ✅ Beijing | ✅ celsius | 0.05 |
| 英文 | `_mix.json` | ✅ Beijing | ✅ celsius | 0.07 |
| 中文 | `_mix.json` | ✅ 北京 | ❌ fahrenheit | **0.00** |

### 智能家居工具

| 输入语言 | 工具文件 | room | brightness | 置信度 |
|----------|---------|------|------------|--------|
| 英文 | `_en.json` | ✅ living room | ✅ 50 | 0.53 |
| 英文 | `_mix.json` | ✅ living room | ✅ 50 | **0.81** |
| 中文 | `_en.json` | ⚠️ living room | ✅ 50 | **0.01** |
| 中文 | `_mix.json` | ✅ 客厅 | ✅ 50 | **0.01** |

---

## 关键发现

### 1. 中文输入置信度归零

```
英文输入: 0.05 - 0.81 (可用)
中文输入: 0.00 - 0.01 (不可用)
```

**无论工具描述是中文、英文还是混合，只要输入是中文，置信度就归零。**

### 2. 工具描述优化效果有限

- 加中文示例 → city 正确了，但 unit 仍错
- 加默认值说明 → 无变化
- 混合描述 → 英文更好，中文更差

### 3. 参数提取可能正确，但置信度无参考价值

```json
{
  "city": "北京",      // ✅ 正确
  "unit": "fahrenheit", // ❌ 错误
  "confidence": 0.00    // 置信度归零
}
```

---

## 复刻验证

### 环境要求

| 平台 | 内存 | GPU | 说明 |
|------|------|-----|------|
| Linux x86_64 | 28MB | 不需要 | x86_64 PC/服务器 ✅ |
| Linux ARM64 | 28MB | 不需要 | 树莓派 3B+ ❌ (SIGILL) |
| macOS ARM64 | 28MB | 不需要 | Apple Silicon ✅ |

### 验证步骤

1. 下载 Needle 2 预编译版
2. 运行英文测试
3. 运行中文测试
4. 对比置信度

### 预期结果

- 英文测试: 置信度 0.05-0.81
- 中文测试: 置信度 0.00-0.01

---

## 文件清单

```
needle2/
├── README.md          ← 本文件
├── LICENSE            ← MIT License
├── CONTRIBUTING.md    ← 贡献指南
├── *.json             ← 6 组工具定义
└── .gitignore         ← 忽略二进制
```

---

## 相关链接

- Needle 2 官方: https://huggingface.co/Cactus-Compute/needle2
- GitHub: https://github.com/cactus-compute/needle
- 论文: arXiv: 2607.18363

---

## 许可证

MIT License