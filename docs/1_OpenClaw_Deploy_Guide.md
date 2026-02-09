# OpenClaw 环境构建与 API 配置

> 本文档覆盖 OpenClaw 的完整部署流程：从选型对比、硬件准备，到安装部署、接入阿里云 Qwen3 大模型、配置 Nginx HTTPS Web UI。
>
> 返回 [项目总览](../README.md) | 下一步：[Workspace 自定义](./4_OpenClaw_Workspace.md) / [Skill 开发](./5_OpenClaw_Skills.md)

---

## 目录

- [1. 为什么选 OpenClaw？](#1-为什么选-openclaw)
- [2. 硬件与系统准备](#2-硬件与系统准备)
- [3. 安装 OpenClaw](#3-安装-openclaw)
- [4. 接入阿里云 Qwen3 大模型](#4-接入阿里云-qwen3-大模型)
- [5. Nginx HTTPS：局域网 Web UI 访问](#5-nginx-https局域网-web-ui-访问)（[独立文档](./3_OpenClaw_Nginx_WebUI.md)）

---

## 1. 为什么选 OpenClaw？

2026 年初，AI 助手平台百花齐放。在对比了多个方案后，我选择了 OpenClaw，原因如下：

| 特性 | OpenClaw | 其他方案（Dify / LobeChat 等） |
|------|----------|-------------------------------|
| **多渠道接入** | 飞书 / Web UI / API 等 | 大多只有 Web UI |
| **工具调用** | 原生支持 `exec`（可运行任意 bash 命令） | 需要额外开发插件 |
| **技能系统** | Markdown 定义 Skill，零代码扩展能力 | 需要写代码或配置 workflow |
| **模型自由** | 接入任意 OpenAI 兼容 API（DashScope、Ollama 等） | 部分锁定特定模型 |
| **部署方式** | `npm install -g` 一条命令，轻量无 Docker 依赖 | 多数需要 Docker Compose |
| **资源占用** | 仅调用云端 API，本地只跑 Node.js 网关（~50MB 内存） | 本地推理需要 GPU |
| **开源** | MIT 协议，完全开放 | 部分功能闭源 |

**核心吸引力**：OpenClaw 的 Skill 系统 + `exec` 工具让 AI 不再只是"会聊天"——它可以读取服务器状态、查温度、看磁盘、甚至执行自定义脚本。它不是玩具，是真正可以干活的 AI 助手。

---

## 2. 硬件与系统准备

> 这一章是为 OpenClaw 准备运行环境。如果你已经有一台 Linux 服务器或 VPS，可以直接跳到 [第 3 章：安装 OpenClaw](#3-安装-openclaw)。

### 我的硬件：Surface Pro 5

| 项目 | 规格 |
|------|------|
| CPU | Intel Core i5-7300U @ 2.60GHz（双核四线程） |
| 内存 | 8GB LPDDR3 |
| 存储 | TOSHIBA NVMe 256GB |
| GPU | Intel HD Graphics 620（核显） |
| 屏幕 | 12.3 英寸 2736×1824 |
| 发布年份 | 2017 年 |

这台机器跑 Windows 10 已经非常吃力——开几个 Chrome 标签页内存就飙到 90%，没有风扇的 Surface 直接歇菜。但跑 OpenClaw 只需要 Node.js 网关 + 网络连接，完全没问题。

### 环境要求

Ubuntu 资源占用远低于 Windows，非常适合当 7×24 小时服务器。我们只需要确保：

- ✅ Ubuntu 22.04 LTS 运行正常
- ✅ SSH 远程管理已配置
- ✅ Node.js v22+ 已安装（`node -v` 确认）
- ✅ 固定局域网 IP 已绑定（如 `192.168.1.100`）

---

## 3. 安装 OpenClaw

这是最激动人心的部分——三条命令就能把 AI 助手跑起来。

我们使用官方提供的 **[OpenClawInstaller](https://github.com/miaoxworld/OpenClawInstaller)** 一键部署工具来完成安装。

> **OpenClawInstaller** 是 OpenClaw 官方维护的命令行安装工具（MIT 协议），主要特性：
>
> - 一键安装部署，自动检测系统环境并安装依赖
> - 交互式配置菜单（`config-menu.sh`），可视化管理模型、渠道等配置
> - 多模型支持：Claude / GPT / Gemini / Qwen / Ollama 等
> - 多渠道接入：Telegram / Discord / WhatsApp / 飞书 / 微信 / Slack
> - 内置 API 连接测试与诊断工具（`openclaw doctor`）
>
> 此外还有桌面版 **[OpenClaw Manager](https://github.com/miaoxworld/openclaw-manager)**（基于 Tauri 2.0 + React），提供图形界面管理。

### 3.1 克隆安装工具

```bash
git clone https://github.com/miaoxworld/OpenClawInstaller.git
cd OpenClawInstaller
chmod +x install.sh config-menu.sh
```

### 3.2 运行安装脚本

```bash
./install.sh
```

安装脚本会自动完成：

1. 检测系统环境并安装依赖
2. 通过 npm 安装 OpenClaw（`npm install -g openclaw`）
3. 引导核心配置（AI 模型、身份信息）
4. 测试 API 连接
5. 启动 OpenClaw Gateway 服务

### 3.3 核心目录结构

安装完成后，所有配置位于 `~/.openclaw/`：

```
~/.openclaw/
├── openclaw.json        # 🔑 核心配置（模型、渠道、网关）
├── env                  # 🔑 环境变量（API Key）
├── workspace/           # 🧠 AI 人设与能力定义
│   ├── SOUL.md          # 人格定义
│   ├── IDENTITY.md      # 身份信息
│   ├── TOOLS.md         # 工具能力备忘（关键！）
│   ├── AGENTS.md        # Agent 行为准则
│   └── USER.md          # 用户信息
└── skills/              # ⚡ 自定义技能目录
    └── system_info/     # 示例 Skill
```

> OpenClaw 的设计哲学：**一切皆文件**。修改 Markdown 就能改变 AI 的人格、能力和行为，不需要写一行代码。

---

## 4. 接入阿里云 Qwen3 大模型

OpenClaw 支持任何 OpenAI 兼容的 API。阿里云 DashScope 的 Qwen3 系列是国内最好的选择之一——免费额度充足，延迟低，模型能力强。

### 4.1 获取 API Key

1. 访问 [阿里云 DashScope 控制台](https://dashscope.console.aliyun.com/)
2. 开通 DashScope 服务（有免费额度）
3. 在「API-KEY 管理」中创建 API Key

### 4.2 配置环境变量

编辑 `~/.openclaw/env`：

```bash
# 阿里云 DashScope (Qwen) — 兼容 OpenAI 格式
export OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
export OPENAI_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
```

> **关键点**：DashScope 提供 `compatible-mode/v1` 端点，OpenClaw 直接把它当 OpenAI Provider 用。

### 4.3 在 openclaw.json 中注册模型

在 `~/.openclaw/openclaw.json` 的 `models.providers` 中添加自定义 Provider：

```json
{
  "models": {
    "providers": {
      "openai-custom": {
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "apiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "models": [
          {
            "id": "qwen3-14b",
            "name": "Qwen3-14B Dense",
            "api": "openai-completions",
            "reasoning": false,
            "input": ["text"],
            "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
            "contextWindow": 131072,
            "maxTokens": 8192
          },
          {
            "id": "qwen3-32b",
            "name": "Qwen3-32B Dense",
            "api": "openai-completions",
            "reasoning": false,
            "input": ["text"],
            "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
            "contextWindow": 131072,
            "maxTokens": 8192
          },
          {
            "id": "qwen3-4b",
            "name": "Qwen3-4B (轻量)",
            "api": "openai-completions",
            "reasoning": false,
            "input": ["text"],
            "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
            "contextWindow": 131072,
            "maxTokens": 8192
          }
        ]
      }
    }
  }
}
```

> **可选模型**：Qwen3 系列还有 `qwen3-235b-a22b`（MoE）、`qwen3-vl-flash`（视觉）、`qwen3-omni-flash`（全模态语音）等。

### 4.4 设置默认模型

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openai-custom/qwen3-14b"
      },
      "models": {
        "openai-custom/qwen3-4b": {},
        "openai-custom/qwen3-14b": {},
        "openai-custom/qwen3-32b": {}
      }
    }
  }
}
```

命令行切换：

```bash
source ~/.openclaw/env
openclaw models set "openai-custom/qwen3-14b"
openclaw models list  # 确认当前模型
```

实际执行结果：

```
$ openclaw models set "openai-custom/qwen3-14b"
Updated ~/.openclaw/openclaw.json
Default model: openai-custom/qwen3-14b

$ openclaw models list
Model                                      Input      Ctx      Local Auth  Tags
openai-custom/qwen3-14b                    text       128k     no    yes   default,configured
openai-custom/qwen3-4b                     text       128k     no    yes   configured
openai-custom/qwen3-32b                    text       128k     no    yes   configured
```

### 4.5 验证连接

```bash
source ~/.openclaw/env
openclaw agent --agent main --message "你好，你是什么模型？"
```

实际执行结果：

```
$ openclaw agent --agent main --message "你好，你是什么模型？"
我是基于阿里云 Qwen3-14B 模型的 AI 助手。需要我帮你做什么吗？
```

返回正常回复，说明 OpenClaw + Qwen3 已经跑通了！

---

## 5. Nginx HTTPS：局域网 Web UI 访问

> 详细内容已独立成文档，详见：**[Nginx HTTPS 配置指南](./3_OpenClaw_Nginx_WebUI.md)**

OpenClaw 自带 **Control UI**（Web 聊天界面），默认监听 `127.0.0.1:18789`。通过 Nginx 反向代理 + 自签名 SSL，可以让局域网内的手机、平板、其他电脑通过 `https://192.168.1.100:7860` 访问。

<p align="center">
  <img src="../images/openclaw_webui_chat.png" alt="OpenClaw Web UI 聊天界面" width="600" />
</p>

---

> 环境部署完成！接下来可以继续：
>
> - [Workspace 自定义指南](./4_OpenClaw_Workspace.md) — 定义 AI 人格、身份和能力
> - [Skill 开发指南](./5_OpenClaw_Skills.md) — 给 AI 增加自定义能力
> - [踩坑记录与时间线](./6_OpenClaw_Troubleshooting_and_Roadmap.md) — 常见问题和部署经验
> - 返回 [项目总览](../README.md)
