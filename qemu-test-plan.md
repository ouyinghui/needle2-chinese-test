---
title: "Needle 2 QEMU 模拟环境验证测试方案"
date: 2026-08-12
tags: [QEMU, 测试方案, Needle2, ARM64, WSL, Win11]
aliases: [qemu-needle2-test-plan]
related: ["[[README.md]]", "[[MEMORY.md]]"]
status: 草案
---

# Needle 2 QEMU 模拟环境验证测试方案

> 日期: 2026-08-12
> 状态: 草案
> 涉及组件: QEMU, Needle 2, ARM64, WSL, Win11

---

## 一、背景

### 1.1 问题

**当前状态:**
- 树莓派 3B+ 运行 Needle 2 工具调用崩溃 (Illegal instruction)
- 原因: 树莓派 3B+ 的 NEON SIMD 指令与 Needle 2 预编译版不兼容

**问题:**
- 无法在树莓派 3B+ 上测试 Needle 2 的四项核心能力
- 需要找到替代环境进行测试

### 1.2 解决方案

**使用 QEMU 模拟 ARM64 环境:**

- **优点:**
  - 模拟完整的 ARM64 环境
  - 支持所有 ARM64 指令 (包括 NEON)
  - 无需额外硬件
  - 测试环境可控
  - **可在 Win11 + WSL 上运行**

- **缺点:**
  - 性能较慢 (10-100 倍)
  - 需要较多内存 (至少 4GB)
  - 需要下载 QEMU 和 ARM64 镜像

### 1.3 测试目标

**验证 Needle 2 在纯 ARM64 环境下的四项核心能力:**

1. **工具调用 (Tool Calling)** - 天气查询
2. **设备控制 (Device Control)** - 智能家居
3. **结构化提取 (Structured Extraction)** - 邮箱提取
4. **端云协作 (Edge-Cloud Collaboration)** - 离线模式 + 工具模式

---

## 二、环境准备

### 2.1 测试环境架构

```
┌─────────────────────────────────────────────────────────┐
│  Windows 11                                             │
│  ┌───────────────────────────────────────────────────┐  │
│  │  WSL2 - Ubuntu 22.04.5                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  QEMU (arm-softmmu)                         │  │  │
│  │  │  ┌───────────────────────────────────────┐  │  │  │
│  │  │  │  Debian 12 ARM64 虚拟机               │  │  │  │
│  │  │  │  ┌─────────────────────────────────┐  │  │  │  │
│  │  │  │  │  Needle 2 测试                   │  │  │  │  │
│  │  │  │  │  - 工具调用                      │  │  │  │  │
│  │  │  │  │  - 设备控制                      │  │  │  │  │
│  │  │  │  │  - 结构化提取                    │  │  │  │  │
│  │  │  │  │  - 端云协作                      │  │  │  │  │
│  │  │  │  └─────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 2.2 宿主机要求

| 项目 | 要求 | 说明 |
|------|------|------|
| 操作系统 | Windows 11 21H2+ | 需要 WSL2 支持 |
| CPU | 支持虚拟化 | Intel VT-x 或 AMD-V |
| 内存 | 16GB+ | WSL + QEMU + 虚拟机 |
| 存储 | 100GB+ SSD | WSL 磁盘 + 虚拟机镜像 |
| GPU | 可选 | 加速 WSLg |

### 2.3 安装 WSL2

**在 Windows 11 上安装 WSL2:**

```powershell
# 1. 以管理员身份打开 PowerShell

# 2. 安装 WSL2
wsl --install

# 3. 重启计算机

# 4. 验证安装
wsl --list --verbose

# 5. 更新 WSL
wsl --update

# 6. 安装 Ubuntu 22.04.5 (如果未安装)
wsl --install -d Ubuntu-22.04
```

### 2.4 安装 QEMU

**在 Ubuntu 22.04.5 (WSL2) 上安装 QEMU:**

```bash
# 1. 更新软件包
sudo apt update && sudo apt upgrade -y

# 2. 安装 QEMU 和 ARM64 模拟器
sudo apt install -y qemu-system-arm qemu-system-misc qemu-utils

# 3. 验证安装
qemu-system-aarch64 --version
qemu-img --version
```

### 2.5 配置 WSL2 资源

**在 Windows PowerShell 中配置 WSL2 内存:**

```powershell
# 1. 编辑 WSL2 配置
notepad $env:USERPROFILE\.wslconfig

# 2. 添加以下内容:
[wsl2]
memory=12GB
processors=6
swap=4GB
```

### 2.6 资源配置

**QEMU 虚拟机配置:**

| 资源 | 建议值 | 说明 |
|------|--------|------|
| CPU | 4 核 | 至少 2 核 |
| 内存 | 4GB | 至少 2GB |
| 存储 | 20GB | 至少 10GB |
| 网络 | 启用 | 需要下载 Needle 2 |

---

## 三、测试环境搭建

### 3.1 获取 ARM64 镜像

**方式 1: 使用 Debian 12 ARM64 云镜像 (推荐)**

```bash
# 在 WSL2 Ubuntu 中执行
cd ~

# 下载 Debian 12 ARM64 云镜像
wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-arm64.qcow2

# 验证下载
ls -lh debian-12-generic-arm64.qcow2
file debian-12-generic-arm64.qcow2
```

**方式 2: 使用 Ubuntu 22.04 ARM64 云镜像**

```bash
# 在 WSL2 Ubuntu 中执行
cd ~

# 下载 Ubuntu 22.04 ARM64 云镜像
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-arm64.img

# 验证下载
ls -lh jammy-server-cloudimg-arm64.img
```

### 3.2 创建 QEMU 虚拟机

**方式 1: 直接使用云镜像 (推荐)**

```bash
# 在 WSL2 Ubuntu 中执行

# 1. 创建镜像目录
mkdir -p ~/qemu-vm

# 2. 复制镜像
cp ~/debian-12-generic-arm64.qcow2 ~/qemu-vm/
cd ~/qemu-vm

# 3. 启动 QEMU (网络模式: user)
qemu-system-aarch64 \
  -m 4096 \
  -cpu cortex-a72 \
  -smp 4 \
  -drive file=debian-12-generic-arm64.qcow2,format=qcow2 \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0 \
  -nographic

# 4. 退出 QEMU (Ctrl+A, X)
```

**方式 2: 图形模式 (需要 WSLg)**

```bash
# 在 WSL2 Ubuntu 中执行

# 1. 安装 VNC 服务器
sudo apt install -y virt-manager virtinst

# 2. 创建图形化虚拟机
virt-install \
  --name debian-arm64 \
  --ram 4096 \
  --vcpus 4 \
  --disk path=debian-12-generic-arm64.qcow2,format=qcow2 \
  --arch aarch64 \
  --cpu cortex-a72 \
  --network network=default \
  --graphics spice \
  --import

# 3. 使用 virt-manager 管理虚拟机
virt-manager
```

### 3.3 安装 Needle 2

**在 QEMU 虚拟机中:**

```bash
# 1. 登录虚拟机 (通过 SSH)
ssh debian@localhost -p 2222
# 默认密码: debian

# 2. 更新系统
sudo apt update && sudo apt upgrade -y

# 3. 安装 wget
sudo apt install -y wget

# 4. 下载 Needle 2
cd ~
wget https://huggingface.co/Cactus-Compute/needle2/resolve/main/linux-arm64/needle

# 5. 赋予执行权限
chmod +x needle

# 6. 验证
file needle
ls -lh needle
```

### 3.4 验证环境

```bash
# 在 QEMU 虚拟机中执行

# 1. 检查 CPU 架构
uname -m
# 预期: aarch64

# 2. 检查 SIMD 支持
cat /proc/cpuinfo | grep -E "asimd|neon" | head -5
# 预期: 显示 asimd/neon 支持

# 3. 检查内存
free -m
# 预期: 至少 2GB

# 4. 检查 Needle 2
./needle --max 100 --prompt "测试" 2>&1 | head -20
```

---

## 四、测试用例

### 4.1 工具调用 (Tool Calling)

**测试目标:** 验证 Needle 2 能否正确调用工具

**工具配置:**

```json
[
  {
    "name": "get_weather",
    "description": "查询指定城市的天气",
    "parameters": {
      "city": {"type": "string"},
      "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
    }
  }
]
```

**测试命令:**

```bash
# 1. 保存工具配置
cat > tools.json << 'EOF'
[
  {
    "name": "get_weather",
    "description": "查询指定城市的天气",
    "parameters": {
      "city": {"type": "string"},
      "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
    }
  }
]
EOF

# 2. 运行测试
./needle --tools tools.json --max 100 --prompt "今天北京天气怎么样" 2>&1
```

**预期结果:**

```json
{
  "type": "call",
  "success": true,
  "function_calls": [
    {
      "name": "get_weather",
      "arguments": {
        "city": "北京",
        "unit": "celsius"
      }
    }
  ],
  "confidence": > 0.8
}
```

**通过标准:**
- 返回 JSON 格式的工具调用
- 工具名称正确 (get_weather)
- 参数正确提取 (city: 北京, unit: celsius)
- 置信度 > 0.8

---

### 4.2 设备控制 (Device Control)

**测试目标:** 验证 Needle 2 能否生成设备控制指令

**工具配置:**

```json
[
  {
    "name": "turn_on_light",
    "description": "打开指定房间的灯",
    "parameters": {
      "room": {"type": "string"},
      "brightness": {"type": "integer", "minimum": 0, "maximum": 100}
    }
  },
  {
    "name": "set_thermostat",
    "description": "设置空调温度",
    "parameters": {
      "temperature": {"type": "number"},
      "mode": {"type": "string", "enum": ["cool", "heat", "auto"]}
    }
  }
]
```

**测试命令:**

```bash
# 1. 保存工具配置
cat > smart_home_tools.json << 'EOF'
[
  {
    "name": "turn_on_light",
    "description": "打开指定房间的灯",
    "parameters": {
      "room": {"type": "string"},
      "brightness": {"type": "integer", "minimum": 0, "maximum": 100}
    }
  },
  {
    "name": "set_thermostat",
    "description": "设置空调温度",
    "parameters": {
      "temperature": {"type": "number"},
      "mode": {"type": "string", "enum": ["cool", "heat", "auto"]}
    }
  }
]
EOF

# 2. 运行测试
./needle --tools smart_home_tools.json --max 100 --prompt "把客厅的灯调到 50% 亮度" 2>&1
```

**预期结果:**

```json
{
  "type": "call",
  "success": true,
  "function_calls": [
    {
      "name": "turn_on_light",
      "arguments": {
        "room": "客厅",
        "brightness": 50
      }
    }
  ],
  "confidence": > 0.8
}
```

**通过标准:**
- 返回 JSON 格式的工具调用
- 工具名称正确 (turn_on_light)
- 参数正确提取 (room: 客厅, brightness: 50)
- 置信度 > 0.8

---

### 4.3 结构化提取 (Structured Extraction)

**测试目标:** 验证 Needle 2 能否从文本中提取结构化数据

**工具配置:**

```json
[
  {
    "name": "extract_email",
    "description": "从文本中提取邮箱地址",
    "parameters": {
      "text": {"type": "string"}
    },
    "returns": {
      "emails": {"type": "array", "items": {"type": "string"}}
    }
  }
]
```

**测试命令:**

```bash
# 1. 保存工具配置
cat > extract_tools.json << 'EOF'
[
  {
    "name": "extract_email",
    "description": "从文本中提取邮箱地址",
    "parameters": {
      "text": {"type": "string"}
    },
    "returns": {
      "emails": {"type": "array", "items": {"type": "string"}}
    }
  }
]
EOF

# 2. 运行测试
./needle --tools extract_tools.json --max 100 --prompt "请从以下文本提取邮箱: 请联系张三 zhangsan@example.com 或李四 lisi@example.org" 2>&1
```

**预期结果:**

```json
{
  "type": "call",
  "success": true,
  "function_calls": [
    {
      "name": "extract_email",
      "arguments": {
        "text": "请联系张三 zhangsan@example.com 或李四 lisi@example.org"
      },
      "result": {
        "emails": ["zhangsan@example.com", "lisi@example.org"]
      }
    }
  ],
  "confidence": > 0.8
}
```

**通过标准:**
- 返回 JSON 格式的工具调用
- 工具名称正确 (extract_email)
- 参数正确提取 (emails: [zhangsan@example.com, lisi@example.org])
- 置信度 > 0.8

---

### 4.4 端云协作 (Edge-Cloud Collaboration)

**测试目标:** 验证 Needle 2 的离线处理和置信度机制

**测试 1: 离线模式 (无工具)**

```bash
# 1. 运行测试
./needle --max 100 --prompt "1 + 1 等于多少？" 2>&1
```

**预期结果:**

```json
{
  "type": "call",
  "success": true,
  "function_calls": [],
  "reasoning": "简单的数学计算",
  "confidence": > 0.9
}
```

**通过标准:**
- 成功运行
- 返回 reasoning
- 置信度 > 0.9

**测试 2: 低置信度处理**

```bash
# 1. 运行测试
./needle --tools tools.json --max 100 --prompt "今天北京天气怎么样" 2>&1

# 2. 如果置信度低，测试云升级
# (实际测试中，需要实现云升级逻辑)
```

**通过标准:**
- 工具调用成功
- 置信度 > 0.8 (高置信度)
- 或置信度 < 0.5 (低置信度，触发云升级)

---

## 五、性能测试

### 5.1 测试指标

| 指标 | 测试方法 | 预期值 |
|------|---------|--------|
| 预填充速度 | `prefill_tps` | 10-50 tokens/秒 |
| 解码速度 | `decode_tps` | 5-20 tokens/秒 |
| 内存占用 | `peak_ram_mb` | < 28MB |
| 工具调用延迟 | 从输入到输出 | < 5 秒 |

### 5.2 测试命令

```bash
# 1. 测试预填充速度
./needle --max 100 --prompt "测试预填充" 2>&1 | grep prefill_tps

# 2. 测试解码速度
./needle --max 200 --prompt "测试解码" 2>&1 | grep decode_tps

# 3. 测试内存占用
./needle --max 100 --prompt "测试内存" 2>&1 | grep peak_ram_mb

# 4. 测试工具调用延迟
time ./needle --tools tools.json --max 100 --prompt "测试工具调用" 2>&1
```

---

## 六、测试计划

### 6.1 时间表

| 阶段 | 任务 | 耗时 | 状态 |
|------|------|------|------|
| 阶段 1 | 安装 WSL2 和 Ubuntu 22.04.5 | 30 分钟 | ⏳ 待开始 |
| 阶段 2 | 安装 QEMU 和 ARM64 镜像 | 30 分钟 | ⏳ 待开始 |
| 阶段 3 | 搭建 QEMU 虚拟机环境 | 30 分钟 | ⏳ 待开始 |
| 阶段 4 | 下载和安装 Needle 2 | 30 分钟 | ⏳ 待开始 |
| 阶段 5 | 测试工具调用 | 30 分钟 | ⏳ 待开始 |
| 阶段 6 | 测试设备控制 | 30 分钟 | ⏳ 待开始 |
| 阶段 7 | 测试结构化提取 | 30 分钟 | ⏳ 待开始 |
| 阶段 8 | 测试端云协作 | 30 分钟 | ⏳ 待开始 |
| 阶段 9 | 性能测试 | 30 分钟 | ⏳ 待开始 |
| 阶段 10 | 编写测试报告 | 1 小时 | ⏳ 待开始 |

**总耗时: 5 小时**

### 6.2 资源需求

| 资源 | 需求 |
|------|------|
| CPU | 6 核 (WSL2 + QEMU) |
| 内存 | 12GB (WSL2) |
| 存储 | 100GB SSD |
| 网络 | 需要 |

---

## 七、风险分析

### 7.1 潜在风险

| 风险 | 影响 | 概率 | 缓解措施 |
|------|------|------|---------|
| WSL2 性能慢 | 测试时间长 | 高 | 预留足够时间 |
| 内存不足 | 测试失败 | 中 | 配置 WSL2 内存上限 12GB |
| 存储不足 | 测试失败 | 中 | 至少 100GB SSD |
| Needle 2 在 QEMU 中崩溃 | 测试失败 | 低 | 检查 QEMU 配置 |

### 7.2 回退方案

**如果 QEMU 测试失败:**

1. **使用 Docker:**
   ```bash
   docker run -it arm64v8/ubuntu:22.04 bash
   ```

2. **使用在线 ARM64 环境:**
   ```bash
   # 使用 onlineGDB ARM64 在线编译器
   ```

3. **升级到树莓派 5:**
   - 官方支持最好
   - 性能最好

---

## 八、测试报告模板

### 8.1 测试报告结构

```
# Needle 2 QEMU 模拟环境测试报告

## 测试环境

| 项目 | 值 |
|------|------|
| 宿主机 | Windows 11 |
| 虚拟化 | WSL2 - Ubuntu 22.04.5 |
| 模拟环境 | QEMU ARM64 |
| CPU | 6 核 (WSL2) |
| 内存 | 12GB (WSL2) |
| 存储 | 100GB SSD |
| 操作系统 | Debian 12 ARM64 |

## 测试结果

### 1. 工具调用

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 天气查询 | ✅/❌ | ... |

### 2. 设备控制

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 智能家居 | ✅/❌ | ... |

### 3. 结构化提取

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 邮箱提取 | ✅/❌ | ... |

### 4. 端云协作

| 测试项 | 结果 | 说明 |
|--------|------|------|
| 离线模式 | ✅/❌ | ... |
| 工具模式 | ✅/❌ | ... |

## 性能测试

| 指标 | 测试值 | 预期值 | 状态 |
|------|--------|--------|------|
| 预填充速度 | X tok/s | 10-50 tok/s | ✅/❌ |
| 解码速度 | X tok/s | 5-20 tok/s | ✅/❌ |
| 内存占用 | X MB | < 28MB | ✅/❌ |

## 结论

测试通过/失败，原因分析。
```

---

## 九、总结

### 9.1 关键要点

1. **QEMU 模拟 ARM64 环境** - 可以测试 Needle 2 在完整 ARM64 环境下的表现
2. **WSL2 支持** - 可以在 Win11 + WSL2 + Ubuntu 22.04.5 上运行
3. **需要足够资源** - WSL2 内存上限 12GB，至少 100GB SSD
4. **性能较慢** - QEMU 模拟比原生 ARM64 慢 10-100 倍
5. **预期通过四项测试** - 在 QEMU 中应该能通过所有四项核心能力测试

### 9.2 下一步

1. **安装 WSL2** - 在 Windows 11 上安装 WSL2 和 Ubuntu 22.04.5
2. **安装 QEMU** - 在 Ubuntu 22.04.5 上安装 QEMU
3. **下载 ARM64 镜像** - Debian 12 ARM64 云镜像
4. **启动 QEMU** - 运行 Debian 12 ARM64 虚拟机
5. **下载 Needle 2** - 从 Hugging Face
6. **执行测试** - 四项核心能力测试
7. **编写报告** - 记录测试结果和分析

---

*文档生成时间: 2026-08-12*
*文档状态: 草案*