---
type: module-spec
version: 1.0
source: src/telegram/
files_analyzed: 40
loc: 7929
last_updated: 2026-02-09
---

# Telegram Spec

## 1. Intent (意图)
Telegram Bot 平台集成模块：
- Telegram Bot API 封装
- Webhook 和长轮询两种接入方式
- 消息收发、编辑、反应
- DM/群组/话题（topics）消息处理
- Bot 访问控制
- 多账号支持

## 2. Interface (接口定义)
- `createTelegramBot(cfg, accountId)` — 创建 Bot 实例
- `createTelegramWebhookCallback(bot)` — 创建 webhook 回调
- `startTelegramWebhook(bot, url)` — 启动 webhook 服务
- `monitorTelegramProvider(cfg, accountId)` — 启动 Bot 监听
- `sendMessageTelegram(params)` — 发送消息
- `reactMessageTelegram(params)` — 发送反应

## 3. Business Logic (业务逻辑)
- **Bot 启动**: 加载 token → 选择 webhook/polling → 注册消息处理器
- **消息上下文**: 判断 DM vs 群组 → 解析话题 thread ID → 路由到 Agent
- **访问控制**: bot-access 配置 → 白名单检查

## 4. Data Structures (数据结构)
- Bot 消息上下文（DM threads, topic threadId）
- 账号配置
- Allowed updates 配置

## 5. Constraints (约束)
- Telegram Bot API 限制
- Webhook 需要 HTTPS URL
- 消息长度限制 4096 字符

## 6. Edge Cases (边界条件)
- Webhook vs polling 切换
- 话题群组的 thread ID 解析
- API 日志记录

## 7. Technical Debt (技术债务)
- 测试文件名嵌入 describe 内容，过长

## 8. Dependencies (依赖)
- **内部**: channels, config, routing, agents
- **外部**: telegraf/grammy（Telegram Bot 框架）
