---
type: module-spec
version: 1.0
source: src/sessions/
files_analyzed: 6
loc: 330
last_updated: 2026-02-09
---

# Sessions Spec

## 1. Intent (意图)
轻量级 Session 策略模块，处理会话级别的覆盖和策略决策：
- 发送策略（allow/deny）决策
- 模型覆盖（per-session model override）
- Thinking level 覆盖
- Session key 解析工具
- Transcript 事件通知

## 2. Interface (接口定义)
- `resolveSendPolicy(params)` → `"allow" | "deny"` — 判断 session 是否允许发送消息
- `applyModelOverrideToSessionEntry(entry)` — 应用模型覆盖
- `applyVerboseOverride(entry)` — 应用 thinking level 覆盖
- `parseAgentSessionKey(key)` — 解析 session key 结构
- `onSessionTranscriptUpdate(callback)` — 监听 transcript 变更

## 3. Business Logic (业务逻辑)
- **发送策略**: 检查 session entry override → 检查全局 config policy → 按 channel/chatType 匹配规则
- **Session key 派生**: 从 session key 推导 channel 和 chatType（如 `discord:group:xxx`）

## 4. Data Structures (数据结构)
- `SessionSendPolicyDecision` = `"allow" | "deny"`
- `SessionEntry` — session 配置条目（来自 config 模块）

## 5. Constraints (约束)
- 纯策略模块，无状态存储
- 依赖 config 模块提供的 session 配置

## 6. Edge Cases (边界条件)
- 未配置 sendPolicy 时默认 allow
- Session key 格式不规范时的容错

## 7. Technical Debt (技术债务)
- 模块很小（330 LOC），部分逻辑可能属于 config 或 routing 模块

## 8. Dependencies (依赖)
- **内部**: config, channels/chat-type
- **外部**: 无
