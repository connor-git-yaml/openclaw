---
type: module-spec
version: 1.0
source: src/channels/
files_analyzed: 77
loc: 9257
last_updated: 2026-02-09
---

# Channels Spec

## 1. Intent (意图)
Channel 抽象层，为所有消息平台（Discord、Telegram、Slack、iMessage、Signal 等）提供统一接口：
- Channel 配置匹配和解析
- 聊天类型（dm/group/channel）分类
- 消息发送目标解析
- 命令门控（哪些命令在哪些 channel 可用）
- @mention 门控
- ACK 反应（已读回执）
- 会话标签管理
- Sender 身份解析
- Channel 插件注册表

## 2. Interface (接口定义)
- `resolveChannelEntryMatch<T>(params)` — 匹配 channel 配置条目（直接/父级/通配符）
- `normalizeChatType(type)` → `"dm" | "group" | "channel"` — 标准化聊天类型
- `listChannelPlugins()` — 列出所有已注册的 channel 插件
- `resolveCommandGating(cfg, channel, command)` — 判断命令是否在某 channel 可用
- `resolveMentionGating(cfg, channel)` — 判断是否需要 @mention 才响应
- `resolveAckReaction(cfg, channel)` — 解析 ACK 反应 emoji
- `resolveConversationLabel(cfg, channel)` — 解析对话标签
- `resolveSenderIdentity(cfg, channel, sender)` — 解析发送者身份

## 3. Business Logic (业务逻辑)
- **Channel 匹配**: 尝试直接匹配 → 父级匹配 → 通配符匹配 → 返回匹配结果和来源
- **Channel slug 标准化**: 小写 + 去除特殊字符 + 连字符分隔
- **命令门控**: 按 channel + chatType 配置允许/禁止的命令列表
- **插件注册**: 每个 channel 平台注册为插件，提供 send/read/react 等统一操作

## 4. Data Structures (数据结构)
- `ChannelEntryMatch<T>` — 匹配结果（entry, key, matchSource）
- `ChannelMatchSource` = `"direct" | "parent" | "wildcard"`
- `ChatType` = `"dm" | "group" | "channel"`
- `ChannelId` — channel 标识符

## 5. Constraints (约束)
- Channel slug 限制为小写字母、数字和连字符
- 支持三级匹配优先级：直接 > 父级 > 通配符

## 6. Edge Cases (边界条件)
- 未配置 channel 时使用默认值
- 通配符 `*` 匹配所有 channel
- 空 channel ID 的处理

## 7. Technical Debt (技术债务)
- plugins/ 子目录包含大量平台特定代码，可能应该与各 channel 模块合并

## 8. Dependencies (依赖)
- **内部**: config
- **外部**: 无直接外部依赖
