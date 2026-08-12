# Needle 2 资源索引与测试指南

> 日期: 2026-08-12  
> 目的: 方便有兴趣的开发者复刻验证

---

## 一、官方资源

| 资源 | 链接 |
|------|------|
| 官网 | https://cactuscompute.com/needle |
| Hugging Face | https://huggingface.co/Cactus-Compute/needle2 |
| GitHub | https://github.com/cactus-compute/needle |
| 论文 | arXiv: 2607.18363 |
| Python 包 | pip install cactus-needle |

---

## 二、预编译二进制下载

从 Hugging Face 仓库下载对应平台版本：

```bash
# Linux ARM64 (树莓派)
curl -L -H "Authorization: Bearer YOUR_TOKEN" \
  https://huggingface.co/Cactus-Compute/needle2/resolve/main/linux-arm64/needle \
  -o needle

# Linux x86_64 (PC/服务器)
curl -L -H "Authorization: Bearer YOUR_TOKEN" \
  https://huggingface.co/Cactus-Compute/needle2/resolve/main/linux-x86_64/needle \
  -o needle

# macOS Apple Silicon
curl -L -H "Authorization: Bearer YOUR_TOKEN" \
  https://huggingface.co/Cactus-Compute/needle2/resolve/main/macos-arm64/needle \
  -o needle

# macOS Intel
curl -L -H "Authorization: Bearer YOUR_TOKEN" \
  https://huggingface.co/Cactus-Compute/needle2/resolve/main/macos-x86_64/needle \
  -o needle

# Windows
curl -L -H "Authorization: Bearer YOUR_TOKEN" \
  https://huggingface.co/Cactus-Compute/needle2/resolve/main/windows-x86_64/needle.exe \
  -o needle.exe
```

**注意:** 需要 Hugging Face Token，可从 https://huggingface.co/settings/tokens 获取。

---

## 三、快速测试

### 3.1 基本测试

```bash
# 查看帮助
./needle --help

# 基本推理（无工具）
./needle --max 200 --prompt "What is 2+2?"

# 查看模型信息
./needle --max 100 --prompt "What model are you?"
```

### 3.2 工具调用测试

**创建工具定义文件 `tools.json`:**

```json
[
  {
    "name": "get_weather",
    "description": "Get the current weather for a city.",
    "parameters": {
      "type": "object",
      "properties": {
        "city": {
          "type": "string",
          "description": "City name, e.g. 'Beijing', 'Shanghai'"
        },
        "unit": {
          "type": "string",
          "enum": ["celsius", "fahrenheit"],
          "description": "Temperature unit. Default: celsius"
        }
      },
      "required": ["city"]
    }
  }
]
```

**执行测试:**

```bash
# 英文测试
./needle --tools tools.json --max 200 --prompt "What's the weather in Beijing?"

# 期望输出:
# {"type":"call","function_calls":[{"name":"get_weather","arguments":{"city":"Beijing","unit":"celsius"}}],"confidence":0.94}

# 中文测试（预期置信度低）
./needle --tools tools.json --prompt "北京天气怎么样？"

# 期望输出:
# {"type":"call","function_calls":[{"name":"get_weather","arguments":{"city":"北京","unit":"fahrenheit"}}],"confidence":0.00}
```

### 3.3 HTTP 服务器模式

```bash
# 启动 HTTP 服务器 (localhost:8080)
./needle --tools tools.json --serve

# 调用测试
curl -X POST "http://localhost:8080/complete" \
  -H "Content-Type: application/json" \
  -d '{"input": "What is the weather in Beijing?"}'
```

---

## 四、对比测试：Needle 2 vs Qwen2.5-3B

### 4.1 Needle 2

```bash
# 下载
# (使用上面的预编译命令)

# 内存占用
./needle --max 100 --prompt "test"
# peak_ram_mb: 28.0

# 速度 (树莓派 5)
# decode_tps: 500
```

### 4.2 Qwen2.5-3B

```bash
# 下载
git clone https://huggingface.co/Qwen/Qwen2.5-3B-Instruct

# vLLM 部署
pip install vllm
vllm serve Qwen/Qwen2.5-3B-Instruct

# 工具调用 (vLLM >= 0.6)
vllm serve Qwen/Qwen2.5-3B-Instruct \
  --enable-auto-tool-choice \
  --tool-call-parser hermes

# 调用
curl -X POST "http://localhost:8000/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen2.5-3B-Instruct",
    "messages": [
      {"role": "user", "content": "北京天气怎么样？"}
    ],
    "tools": [...]
  }'
```

---

## 五、测试结果汇总

### Needle 2

| 输入 | city | unit | 置信度 |
|------|------|------|--------|
| What's the weather in Beijing? | ✅ Beijing | ✅ celsius | 0.05 |
| 北京天气怎么样？ | ✅ 北京 | ❌ fahrenheit | 0.00 |

### 关键发现

- Needle 2 对英文工具调用支持良好
- 中文输入置信度归零 (0.00)
- 参数提取可能正确，但置信度无参考价值

---

## 六、环境要求

### Needle 2

| 平台 | 架构 | 内存 | GPU |
|------|------|------|-----|
| Linux ARM64 | aarch64 | 28MB | 不需要 |
| Linux x86_64 | amd64 | 28MB | 不需要 |
| macOS ARM64 | arm64 | 28MB | 不需要 |
| Windows x64 | amd64 | 28MB | 不需要 |

### Qwen2.5-3B

| 平台 | 内存 | GPU |
|------|------|-----|
| CPU | ~6GB | 不需要 |
| GPU | ~4GB VRAM | 推荐 |

---

## 七、常见问题

### Q1: Needle 2 支持中文吗？

A: 官方未明确说明，但测试结果显示中文置信度为 0。建议仅用于英文场景。

### Q2: 如何获取 Hugging Face Token？

A: 访问 https://huggingface.co/settings/tokens 创建个人访问令牌。

### Q3: Needle 2 可以微调吗？

A: 支持微调，但需要完整训练代码和训练数据，成本较高。

### Q4: 中文工具调用推荐什么模型？

A: Qwen2.5-3B 或更大版本，原生支持中文工具调用。

---

## 八、许可证

- Needle 2: Apache 2.0
- Qwen2.5: Apache 2.0

---

## 九、联系方式

- Cactus-Compute: https://cactuscompute.com
- Needle GitHub: https://github.com/cactus-compute/needle
- Discord: (官网提供)

---

## 版本记录

| 版本 | 日期 | 内容 |
|------|------|------|
| v1.0 | 2026-08-12 | 初版，资源索引与测试指南 |