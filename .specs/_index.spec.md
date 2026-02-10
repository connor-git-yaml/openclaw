---
type: architecture-overview
version: 1.1
total_modules: 59+
total_files: 2618
total_loc: ~460000
last_updated: 2026-02-10
---

# OpenClaw Architecture Overview

## 项目简介
OpenClaw 是一个 AI Agent 网关平台，将 LLM（Claude、GPT、Gemini 等）连接到多种消息平台（Discord、Telegram、Slack、iMessage 等），提供完整的 Agent 生命周期管理。

## 架构层次

```
┌─────────────────────────────────────────────────┐
│                   CLI / TUI                      │
│              (cli, tui, commands)                 │
├─────────────────────────────────────────────────┤
│                   Gateway                        │
│         (WebSocket/HTTP JSON-RPC Server)         │
├──────────┬──────────┬──────────┬────────────────┤
│  Agents  │ Channels │ Routing  │   Sessions     │
│          │          │          │                │
├──────────┴──────────┴──────────┴────────────────┤
│              Platform Integrations               │
│    (discord, telegram, slack, imessage, signal)  │
├─────────────────────────────────────────────────┤
│              Feature Modules                     │
│  (browser, cron, memory, hooks, media, tts)     │
├─────────────────────────────────────────────────┤
│              Infrastructure                      │
│  (config, infra, logging, security, process)    │
└─────────────────────────────────────────────────┘
```

## 模块清单

### Tier 1 — 核心模块

| 模块 | LOC | 文件数 | 职责 |
|------|-----|--------|------|
| [gateway](gateway.spec.md) | 26K | 133 | WebSocket/HTTP 网关服务器 |
| [agents](agents.spec.md) | 47K | 233 | AI Agent 生命周期管理 |
| [config](config.spec.md) | 14K | 89 | 配置系统（JSON5 + Zod） |
| [channels](channels.spec.md) | 9K | 77 | 消息平台抽象层 |
| [routing](routing.spec.md) | 646 | 3 | 消息路由和绑定 |
| [sessions](sessions.spec.md) | 330 | 6 | 会话策略管理 |
| [providers](providers.spec.md) | 411 | 4 | LLM 提供商认证 |

### Tier 2 — 功能模块

| 模块 | LOC | 文件数 | 职责 |
|------|-----|--------|------|
| [commands](commands.spec.md) | 29K | 174 | CLI 命令实现 |
| [browser](browser.spec.md) | 10K | 52 | Chrome CDP 浏览器自动化 |
| [exec](exec.spec.md) | 8K | 35 | 安全进程执行和管理 |
| [nodes](nodes.spec.md) | 12K | 48 | 分布式节点管理和通信 |
| [canvas](canvas.spec.md) | 6K | 28 | 动态 Web 内容展示和 A2UI |
| [messaging](messaging.spec.md) | 15K | 67 | 跨平台消息系统核心 |
| [memory](memory.spec.md) | 7K | 28 | 向量记忆搜索 |
| [discord](discord.spec.md) | 9K | 44 | Discord Bot 集成 |
| [telegram](telegram.spec.md) | 8K | 40 | Telegram Bot 集成 |
| [slack](slack.spec.md) | 6K | 43 | Slack App 集成 |
| [heartbeat](heartbeat.spec.md) | 5K | 21 | 代理健康监控和周期任务 |
| [web-fetch](web-fetch.spec.md) | 4K | 18 | 轻量级网页内容抓取 |
| [web-search](web-search.spec.md) | 5K | 22 | Brave API 网页搜索 |
| [tts](tts.spec.md) | 7K | 31 | 多提供商文本转语音 |
| [image](image.spec.md) | 9K | 42 | 计算机视觉和图像分析 |
| [cron](cron.spec.md) | 4K | 22 | 定时任务调度 |
| [hooks](hooks.spec.md) | 4K | 22 | 事件驱动自动化 |
| [media](media.spec.md) | 2K | 12 | 媒体文件处理 |

### Tier 3 — 工具模块

| 模块 | LOC | 文件数 | 职责 |
|------|-----|--------|------|
| [infra](infra.spec.md) | 23K | 121 | 基础设施（事件、发现、TLS） |
| [cli](cli.spec.md) | 21K | 138 | 命令行参数解析 |
| [tui](tui.spec.md) | 4K | 24 | 终端交互界面 |
| [security](security.spec.md) | 4K | 8 | 安全审计 |
| [logging](logging.spec.md) | 2K | 10 | 日志子系统 |
| [process](process.spec.md) | 513 | 5 | 子进程管理 |

### Tier 4 — 其他模块（待详细文档化）

| 模块 | 职责 |
|------|------|
| imessage | iMessage 集成 |
| signal | Signal 集成 |
| line | LINE 集成 |
| whatsapp | WhatsApp Business 集成 |
| matrix | Matrix 去中心化消息 |
| macos | macOS 原生集成 |
| acp | Agent Communication Protocol |
| pairing | 设备配对 |
| plugin-sdk | 插件 SDK |
| plugins | 插件加载器 |
| terminal | 终端工具 |
| auto-reply | 自动回复策略 |
| link-understanding | 链接内容理解 |
| media-understanding | 媒体内容理解 |
| markdown | Markdown 处理 |
| shared | 共享工具 |
| daemon | 守护进程管理 |
| wizard | 引导向导 |
| compat | 兼容层 |

## 核心数据流

```
User Message (Discord/Telegram/CLI/...)
    ↓
Channel Plugin (discord/telegram/slack)
    ↓
Gateway (WebSocket JSON-RPC)
    ↓
Routing (resolve agent + session key)
    ↓
Agent (build context + call LLM)
    ↓
Tool Execution (bash, browser, message, etc.)
    ↓
Response → Channel Plugin → User
```

## 关键设计决策
1. **JSON-RPC over WebSocket** — Gateway 使用 JSON-RPC 2.0 协议，支持双向通信
2. **Agent 隔离** — 每个 Agent 有独立的 workspace、session、skills
3. **多 Provider** — 支持 Anthropic、OpenAI、Google、Bedrock、Copilot 等多个 LLM
4. **插件化 Channel** — 每个消息平台是独立插件，通过统一接口接入
5. **本地优先** — 配置和数据存储在本地文件系统（~/.openclaw/）
6. **向量记忆** — 基于 SQLite + sqlite-vec 的本地向量搜索
