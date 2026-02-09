# OpenClaw 自定义 Workspace：让 AI 有灵魂

> 本文档从 [README 主文档](../README.md) 中独立出来，详细介绍如何通过 Workspace 文件定义 AI 的人格、身份和能力边界。
>
> 相关文档：[Skill 开发指南](./5_OpenClaw_Skills.md)

---

## 目录

- [1. Workspace 概述](#1-workspace-概述)
- [2. SOUL.md — 定义人格](#2-soulmd--定义人格)
- [3. IDENTITY.md — 身份卡片](#3-identitymd--身份卡片)
- [4. TOOLS.md — 工具能力备忘（关键文件！）](#4-toolsmd--工具能力备忘关键文件)
- [5. 记忆系统 — MEMORY.md + 每日日志](#5-记忆系统--memorymd--每日日志)

---

## 1. Workspace 概述

OpenClaw 最独特的设计之一就是 **Workspace 文件系统**——通过几个 Markdown 文件，你可以完全定义 AI 的人格、知识和能力边界。这不是简单的 system prompt，而是一套结构化的 AI 行为框架。

所有 Workspace 文件位于 `~/.openclaw/workspace/` 目录下：

```
~/.openclaw/workspace/
├── SOUL.md          # 人格定义（性格、规则）
├── IDENTITY.md      # 身份卡片（名字、语言、时区）
├── TOOLS.md         # 工具能力备忘（让 AI 知道自己能做什么）
├── MEMORY.md        # 长期记忆（AI 跨会话记住的信息）
├── memory/          # 每日日志目录
│   └── 2026-02-08.md
├── AGENTS.md        # Agent 行为准则
└── USER.md          # 用户信息
```

## 2. SOUL.md — 定义人格

这是 AI 的"灵魂"，决定了它怎么说话、怎么思考：

```markdown
# 我是谁

我是 cw 的私人AI助手，基于阿里云 Qwen3 系列模型运行，默认使用 qwen3-14b。
通过 OpenClaw 平台运行，支持飞书和网页端聊天。

## 性格
- 友好、简洁、实用
- 用中文回复
- 不啰嗦，直接给出答案

## 能力
- 可以用 `exec` 工具执行 bash 命令，读取本机硬件/软件信息
- 当用户问电脑配置、品牌、温度、磁盘等问题时，直接执行命令获取真实数据
- 参考 TOOLS.md 中的命令速查表

## 规则
- 诚实回答，不确定的事情就说不确定
- 涉及隐私信息时要谨慎
- 能用工具获取真实数据时，不要给"请自行查询"的回答
```

## 3. IDENTITY.md — 身份卡片

```markdown
名字：AI 助手
主人：cw
语言：中文
当前模型：qwen3-14b
时区：Asia/Shanghai
```

## 4. TOOLS.md — 工具能力备忘（关键文件！）

这是让 AI 知道"**自己能做什么**"的核心文件。OpenClaw 在每次会话开始时会读取 `TOOLS.md`，如果里面没有提及某种能力，AI 就不知道自己可以做。

```markdown
# TOOLS.md - 本地工具备忘

## 系统信息采集 (system_info skill)

我可以直接读取这台电脑的硬件和软件信息。

### 一键完整报告
bash ~/.openclaw/skills/system_info/gather_info.sh

### 常用单项命令
- 电脑品牌/型号: cat /sys/devices/virtual/dmi/id/board_vendor
- CPU 型号: grep -m1 'model name' /proc/cpuinfo
- 内存: free -h
- 磁盘: df -h
- 实时温度: sensors
- 网络 IP: hostname -I

## 本机信息速查
- 设备: Microsoft Surface Pro
- CPU: Intel Core i5-7300U @ 2.60GHz
- 内存: 8GB
- 系统: Ubuntu 22.04.3 LTS
```

> **设计巧思**：`TOOLS.md` 底部的"本机信息速查"是一个双保险——即使模型不主动调用 `exec` 工具，它也能直接从这里读取基本信息回答用户。这对小模型尤其有用。

---

## 5. 记忆系统 — MEMORY.md + 每日日志

OpenClaw 内置了**持久记忆**能力——AI 可以跨会话、跨渠道记住用户的信息和偏好。记忆分为两层：

### 5.1 MEMORY.md — 长期记忆

位于 `~/.openclaw/workspace/MEMORY.md`，AI 在每次会话开始时会读取这个文件。你可以手动编辑，也可以让 AI 在对话中自行更新。

建议记录的内容：

```markdown
# MEMORY.md - 长期记忆

> 最后更新：2026-02-08

## 关于主人
- 使用 Surface Pro 5，运行 Ubuntu 22.04 LTS
- 局域网 IP：192.168.1.100
- 偏好用中文交流，喜欢简洁直接的回答

## 系统环境
- OS: Ubuntu 22.04.3 LTS
- Node.js: v22+
- Nginx: 反向代理 OpenClaw Web UI，端口 7860

## OpenClaw 配置
- 默认模型: qwen3-14b
- 渠道: 飞书 + Web UI
- Skill: system_info

## 重要踩坑记录
1. NO_PROXY 必须包含 dashscope.aliyuncs.com，否则阿里云 API 走代理会超时
2. Nginx /ws location 要在 proxy_pass 里注入 token
3. 新 Skill 创建后必须 openclaw gateway --force 重启
```

> **注意**：`MEMORY.md` 中不要放敏感信息（密码、API Key 等），因为它会被注入到每次 AI 会话的上下文中。

### 5.2 每日日志 — memory/ 目录

位于 `~/.openclaw/workspace/memory/` 目录下，按日期命名（如 `2026-02-08.md`）。OpenClaw 会自动管理这些日志，记录每天的对话摘要和重要事件。

```
~/.openclaw/workspace/memory/
└── 2026-02-08.md    # 当天的操作日志
```

日志示例：

```markdown
# 2026-02-08 日志

## 今日完成
### OpenClaw 部署（17:00 - 22:00）
- 安装 OpenClaw，配置阿里云 DashScope API
- Nginx HTTPS 反向代理（端口 7860）
- 开发 system_info Skill
- 测试 Qwen3 模型：4B → 14B（推荐）→ 32B

## 待完成
- [ ] 更多 Skill 开发
- [ ] linux-surface 内核安装
```

### 5.3 记忆系统的作用

| 场景 | 没有记忆 | 有记忆 |
|------|----------|--------|
| 问"我的电脑是什么" | 需要重新查询 | 直接从 MEMORY.md 读取 |
| 问"上次说的那个问题" | 不知道在说什么 | 可以回顾每日日志 |
| 切换飞书↔Web UI | 上下文丢失 | 跨渠道共享记忆 |
| 重启 Gateway | 对话历史清空 | 长期记忆保留 |

> **最佳实践**：初次部署后，花 5 分钟手动编辑 `MEMORY.md`，写入基本的用户信息和系统环境。这样 AI 从第一次对话就能"认识你"。

