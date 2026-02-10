---
type: module-spec
version: 1.0
source: src/agents/
files_analyzed: 233
loc: 46996
last_updated: 2026-02-09
---

# Agents Spec

## 1. Intent (意图)
Agents 是 OpenClaw 最大的模块，负责 AI Agent 的完整生命周期管理：
- Agent 配置解析和作用域管理
- 与 LLM 提供商的交互（Anthropic Claude、OpenAI、Google Gemini、Bedrock 等）
- 工具调用执行（bash、文件操作、浏览器、消息发送等）
- Session/对话管理和上下文窗口控制
- 模型选择、fallback、认证配置
- Subagent 注册和管理
- Skills（技能）加载和刷新

## 2. Interface (接口定义)
- `resolveAgentWorkspaceDir(cfg, agentId)` — 解析 Agent 工作目录
- `listAgentIds(cfg)` — 列出所有配置的 Agent ID
- `resolveDefaultAgentId(cfg)` — 解析默认 Agent
- `runCliAgent(opts)` — 执行 CLI Agent 交互（核心入口）
- `runEmbeddedPiAgent(opts)` — 嵌入式 Pi Agent 运行
- `buildWorkspaceSkillSnapshot()` — 构建工作区技能快照
- `AgentScope` / `ResolvedAgentConfig` — Agent 配置类型
- `ensureAuthProfileStore()` — 认证 profile 管理
- `loadModelCatalog()` — 加载模型目录
- `runWithModelFallback()` — 带 fallback 的模型调用

## 3. Business Logic (业务逻辑)
- **Agent 解析**: 从配置文件解析 agent 列表 → 按 ID 匹配 → 解析 workspace/skills/model
- **模型选择**: 配置模型 → 检查 provider → 应用 thinking level → fallback 链
- **工具执行**: bash 命令（带 PTY 支持）、文件读写、apply-patch、浏览器操作
- **上下文窗口**: 监控 token 使用 → 自动压缩（compaction）→ 上下文窗口保护
- **Skills**: 扫描 workspace 和远程节点 → 加载 SKILL.md → 注入到系统提示
- **Subagent**: 注册子 Agent → 隔离 session → 结果回传

## 4. Data Structures (数据结构)
- `ResolvedAgentConfig` — 完整的 Agent 配置（name, workspace, model, skills, heartbeat, identity 等）
- `AgentEntry` — 配置文件中的 Agent 条目
- `AgentRunContext` — 运行时上下文（sessionKey, verboseLevel, isHeartbeat）
- `BashProcessRegistry` — 管理运行中的 bash 进程
- Model catalog 缓存

## 5. Constraints (约束)
- Agent ID 限制为 `[a-z0-9][a-z0-9_-]{0,63}`
- 默认 Agent ID 为 "main"
- 支持多 provider: anthropic, openai, google, bedrock, copilot, chutes 等
- Skills 支持本地和远程（通过 Node）

## 6. Edge Cases (边界条件)
- 未配置 Agent 时使用默认 "main" Agent
- 认证 token 过期自动刷新
- 模型不可用时的 fallback 链
- 上下文窗口溢出时的自动压缩
- CLI Agent（Claude CLI）不可用时的错误处理

## 7. Technical Debt (技术债务)
- 模块过大（233 个文件，47K LOC），应该拆分为子模块
- bash-tools 相关文件过多，工具系统可抽象为插件架构
- 多个认证方式散落各处（auth-profiles, cli-credentials, oauth 等）
- Pi/Claude CLI 集成代码与核心 Agent 逻辑耦合

## 8. Dependencies (依赖)
- **内部**: config, routing, sessions, infra, logging, browser, memory, channels, security
- **外部**: @anthropic-ai/sdk, openai, ws, chokidar
