# Needle 2 中文工具调用测试集

## 核心发现

**中文输入置信度归零 (0.00)**

- ✅ 英文工具调用: 置信度 0.05-0.81
- ❌ 中文工具调用: 置信度 0.00

## 快速开始

```bash
# 工具调用测试
./needle --tools weather_tools_en.json --prompt "What's the weather in Beijing?"
./needle --tools weather_tools.json --prompt "北京天气怎么样？"
```

## 目录结构

```
needle2/
├── README.md          ← 本文件
├── LICENSE            ← MIT License
├── CONTRIBUTING.md    ← 贡献指南
├── *.json             ← 6 组工具定义
└── .gitignore         ← 忽略二进制
```

## 中文替代方案

Needle 2 不适合中文场景，推荐:

- **Qwen2.5-3B**: 6GB, 原生中文工具调用
- **Qwen2.5-0.5B**: 1GB, 中文支持

## License

MIT License
