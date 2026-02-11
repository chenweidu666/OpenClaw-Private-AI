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

我是 cw 的私人AI助手，基于 Qwen3-14B 模型运行（DashScope 云端 API）。
通过 OpenClaw 平台运行（NAS Docker 容器），支持飞书聊天。

## 性格
- 友好、简洁、实用
- 用中文回复
- 不啰嗦，直接给出答案

## 能力
- 使用 OpenClaw 内置的 23 个核心工具（文件读写、Shell 执行、网页搜索等）
- 可以用 `exec` 工具执行 bash 命令
- 参考 TOOLS.md 中的工具速查

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
当前模型：Qwen3-14B（DashScope 云端 API）
部署：NAS Docker 容器 → 飞书 WebSocket
时区：Asia/Shanghai
```

## 4. TOOLS.md — 工具能力备忘（关键文件！）

这是让 AI 知道"**自己能做什么**"的核心文件。OpenClaw 在每次会话开始时会读取 `TOOLS.md`，如果里面没有提及某种能力，AI 就不知道自己可以做。

```markdown
# TOOLS.md - 环境与工具速查

## 工具说明
当前使用 OpenClaw 内置的 23 个核心工具（文件读写、Shell 执行、网页搜索、定时任务、记忆、飞书消息等）。
无自定义 Function Calling 工具（已于 2026-02-11 清理，精简系统提示词）。

## NAS 存储（容器内可直接访问）
| 容器路径 | 内容 |
|----------|------|
| /nas/volume2/Movies | 电影库 |
| /nas/volume2/Photos | 照片 |
| /nas/volume2/Musics | 音乐 |
| /nas/volume2/Games  | 游戏 |

常用搜索：exec → find /nas/volume2/Movies -iname "*关键词*" -type f

## SSH 主机与设备清单
| 别名 | IP | 设备 | 用途 |
|------|-----|------|------|
| nas  | 192.168.31.10  | 绿联 DH4300 Plus      | OpenClaw 宿主机 + 存储 |
```

> **设计变更**（2026-02-11）：精简系统提示词后，TOOLS.md 从详细的自定义工具列表精简为核心环境信息，包含 NAS 存储路径映射便于 AI 访问媒体目录。当前使用云端 Qwen3-14B（131K 上下文），但保持精简的设计更加聚焦实用。

---

## 5. 记忆系统 — MEMORY.md + 每日日志

OpenClaw 内置了**持久记忆**能力——AI 可以跨会话、跨渠道记住用户的信息和偏好。记忆分为两层：

### 5.1 MEMORY.md — 长期记忆

位于 `~/.openclaw/workspace/MEMORY.md`，AI 在每次会话开始时会读取这个文件。你可以手动编辑，也可以让 AI 在对话中自行更新。

建议记录的内容：

```markdown
# MEMORY.md - 长期记忆

> 最后更新：2026-02-11

## 关于主人
- 偏好用中文交流，喜欢简洁直接的回答

## 系统环境
- 运行在 NAS Docker 容器中
- NAS: 绿联 DH4300+ (RK3588C / 8GB / Debian 12)
- 模型: DashScope Qwen3-14B（云端 API）

## OpenClaw 配置
- 主力模型: Qwen3-14B（DashScope 云端 API）
- 可选升级: Qwen3-32B（DashScope）
- 渠道: 飞书 WebSocket
- 工具: 23 个内置工具（无自定义工具）

## 重要记录
1. 纯云端 API 推理，无需本地 GPU
2. 启动后自动发飞书通知
3. 上下文窗口 131K tokens
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

