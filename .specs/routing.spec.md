---
type: module-spec
version: 1.0
source: src/routing/
files_analyzed: 3
loc: 646
last_updated: 2026-02-09
---

# Routing Spec

## 1. Intent (意图)
消息路由模块，决定传入消息应该由哪个 Agent 处理：
- 基于 bindings 配置将 channel/peer/guild 映射到 Agent
- Session key 的构建、解析和标准化
- Agent 绑定规则匹配

## 2. Interface (接口定义)
- `resolveAgentRoute(input: ResolveAgentRouteInput): ResolvedAgentRoute` — 核心路由解析
- `listBindings(cfg)` — 列出所有绑定规则
- `buildAgentSessionKey(params)` — 构建 session key
- `normalizeAgentId(value)` — 标准化 Agent ID
- `parseAgentSessionKey(key)` — 解析 session key 结构
- `DEFAULT_AGENT_ID = "main"`, `DEFAULT_ACCOUNT_ID = "default"`

## 3. Business Logic (业务逻辑)
- **路由匹配优先级**: peer 绑定 > 父 peer 绑定 > guild 绑定 > team 绑定 > account 绑定 > channel 绑定 > 默认 Agent
- **Session key 格式**: `agent:<agentId>:<rest>` — 如 `agent:main:discord:group:12345`
- **绑定规则**: 配置文件中的 `bindings` 数组，每条规则指定 match 条件和目标 agentId

## 4. Data Structures (数据结构)
- `ResolveAgentRouteInput` — 路由输入（cfg, channel, accountId, peer, parentPeer, guildId, teamId）
- `ResolvedAgentRoute` — 路由结果（agentId, channel, accountId, sessionKey, mainSessionKey, matchedBy）
- `AgentBinding` — 绑定规则（来自 config）
- `SessionKeyShape` = `"missing" | "agent" | "legacy_or_alias" | "malformed_agent"`

## 5. Constraints (约束)
- Agent ID 限制为 `[a-z0-9][a-z0-9_-]{0,63}`
- Session key 必须以 `agent:` 前缀开头

## 6. Edge Cases (边界条件)
- 无匹配绑定时回退到默认 Agent
- 通配符 `*` accountId 匹配所有账户
- 遗留 session key 格式兼容

## 7. Technical Debt (技术债务)
- 模块很小但与 sessions/session-key-utils 有循环依赖风险

## 8. Dependencies (依赖)
- **内部**: config, agents/agent-scope, channels/chat-type, sessions/session-key-utils
- **外部**: 无
