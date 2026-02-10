---
type: module-spec
version: 1.0
source: src/discord/
files_analyzed: 44
loc: 8733
last_updated: 2026-02-09
---

# Discord Spec

## 1. Intent (意图)
Discord 平台集成模块：
- Discord Bot 连接和事件监听
- 消息收发、反应、编辑、删除
- 频道和线程管理
- Gateway logging（Discord WebSocket 事件日志）
- 消息分块（超长消息自动分割）
- Discord API 封装
- 审计日志

## 2. Interface (接口定义)
- `monitorDiscordProvider(cfg, accountId)` — 启动 Discord Bot 监听
- `sendMessageDiscord(params)` — 发送消息
- `sendPollDiscord(params)` — 发送投票
- Discord API 封装（channel CRUD, member info, role info, emoji, sticker 等）
- 消息分块: `chunkMessage(text, limit)` — 智能分割长消息

## 3. Business Logic (业务逻辑)
- **Bot 启动**: 加载 token → 连接 Discord Gateway → 注册事件处理器
- **消息处理**: 接收消息 → 检查 mention/allowlist → 路由到 Agent → 回复
- **分块**: 按 Discord 2000 字符限制 → 优先在换行/代码块边界分割
- **多账号**: 支持多个 Discord Bot 账号

## 4. Data Structures (数据结构)
- Discord 消息、频道、成员等类型（来自 discord.js）
- 账号配置

## 5. Constraints (约束)
- Discord API 速率限制
- 消息长度限制 2000 字符
- 需要 Bot token

## 6. Edge Cases (边界条件)
- Bot 断线重连
- 速率限制处理
- 超长消息分块边界

## 7. Technical Debt (技术债务)
- gateway-logging 命名可能与主 gateway 模块混淆

## 8. Dependencies (依赖)
- **内部**: channels, config, routing, agents
- **外部**: discord.js
