---
type: module-spec
version: 1.0
source: src/slack/
files_analyzed: 43
loc: 5809
last_updated: 2026-02-09
---

# Slack Spec

## 1. Intent (意图)
Slack 平台集成模块：
- Slack Bot/App 连接和事件监听
- 消息 CRUD、反应、pin、emoji 等全套操作
- 频道迁移工具
- Slack 格式化（mrkdwn）
- 多账号/workspace 支持
- Token 解析（bot token + app token）

## 2. Interface (接口定义)
- `monitorSlackProvider(cfg, accountId)` — 启动 Slack 监听
- `sendSlackMessage(params)` / `sendMessageSlack(params)` — 发送消息
- `readSlackMessages(params)` — 读取消息
- `reactSlackMessage(params)` — 发送反应
- `listSlackEmojis()` / `listSlackPins()` — 列表操作
- `probeSlack(token)` — 探测 Slack 连接
- `resolveSlackBotToken(cfg, accountId)` — 解析 Bot token

## 3. Business Logic (业务逻辑)
- **连接**: Socket Mode（app token）或 HTTP events → 消息处理 → 路由到 Agent
- **格式化**: Markdown → Slack mrkdwn 转换
- **频道迁移**: 旧频道 ID → 新频道 ID 映射

## 4. Data Structures (数据结构)
- Slack 消息、频道、成员类型
- 账号配置

## 5. Constraints (约束)
- Slack API 速率限制
- 需要 Bot OAuth token + App-level token
- 消息长度限制

## 6. Edge Cases (边界条件)
- Token 失效重新认证
- 频道 ID 变更迁移
- emoji 自定义 vs 标准

## 7. Technical Debt (技术债务)
- 两个发送函数命名不一致（sendSlackMessage vs sendMessageSlack）

## 8. Dependencies (依赖)
- **内部**: channels, config, routing
- **外部**: @slack/bolt 或 @slack/web-api
