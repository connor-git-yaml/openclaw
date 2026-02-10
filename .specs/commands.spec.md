---
type: module-spec
version: 1.0
source: src/commands/
files_analyzed: 174
loc: 28567
last_updated: 2026-02-09
---

# Commands Spec

## 1. Intent (意图)
CLI 命令实现模块，提供所有 `openclaw` 子命令的业务逻辑：
- `agent` — 执行 Agent 对话（核心命令）
- `agents` — Agent 管理（add/remove/list/bindings/commands）
- `gateway` — Gateway 服务管理（start/stop/restart/status）
- `models` — 模型管理（list/add/set）
- `config` — 配置管理
- `browser` — 浏览器控制
- `cron` — 定时任务管理
- `memory` — 记忆索引管理
- `security` — 安全审计
- `onboard` — 初始化引导
- 以及 hook, skill, node, session 等更多命令

## 2. Interface (接口定义)
- `agentCommand(opts, runtime, deps)` — 执行 Agent 命令（最核心入口）
- `deliverAgentCommandResult(result, opts)` — 投递 Agent 结果到 channel
- 每个子命令导出对应的处理函数
- `CliDeps` — CLI 依赖注入接口

## 3. Business Logic (业务逻辑)
- **Agent 命令**: 解析 session key → 加载模型配置 → 构建 skill snapshot → 运行 Agent → 投递结果
- **Gateway 命令**: 通过 daemon 管理 gateway 进程生命周期
- **Model fallback**: 主模型失败 → 尝试 fallback 列表 → 返回第一个成功结果
- **Onboarding**: 交互式引导 → 选择 provider → 配置 API key → 写入配置

## 4. Data Structures (数据结构)
- `AgentCommandOpts` — Agent 命令参数（message, sessionKey, deliver, model 等）
- `CliDeps` — CLI 依赖（config loader, gateway client 等）

## 5. Constraints (约束)
- 命令实现与 CLI 参数解析分离（CLI 解析在 cli 模块）
- 所有命令共享 CliDeps 注入

## 6. Edge Cases (边界条件)
- Gateway 未启动时的自动启动
- Agent 超时处理
- 模型不可用时的 fallback

## 7. Technical Debt (技术债务)
- 174 个文件，模块过大，应按命令分组为子目录
- 部分命令文件命名不一致

## 8. Dependencies (依赖)
- **内部**: agents, config, gateway, routing, sessions, infra, logging, browser, memory, cron, security, hooks
- **外部**: @clack/prompts（交互式 UI）
