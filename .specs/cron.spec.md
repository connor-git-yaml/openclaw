---
type: module-spec
version: 1.0
source: src/cron/
files_analyzed: 22
loc: 3767
last_updated: 2026-02-09
---

# Cron Spec

## 1. Intent (意图)
定时任务调度模块，支持：
- Cron 表达式解析和调度
- 隔离 Agent 运行（独立 session 的定时 Agent 调用）
- 结果投递到指定 channel/target
- 自然语言时间解析

## 2. Interface (接口定义)
- `runCronIsolatedAgentTurn(opts)` → `RunCronAgentTurnResult` — 执行一次隔离的 Agent 调用
- `parseCronExpression(expr)` — 解析 cron 表达式
- `normalizeCronSchedule(schedule)` — 标准化调度配置
- `deliverCronResult(result, target)` — 投递执行结果

## 3. Business Logic (业务逻辑)
- **隔离执行**: 创建独立 session → 运行 Agent → 收集输出 → 投递结果
- **结果投递**: 提取最后一条非空 Agent 文本 → 通过 message tool 发送到目标 channel
- **HEARTBEAT_OK 处理**: 如果 Agent 回复 HEARTBEAT_OK，跳过投递

## 4. Data Structures (数据结构)
- `RunCronAgentTurnResult` — 执行结果
- `CronConfig` — cron 配置（schedule, agent, target, prompt 等）

## 5. Constraints (约束)
- 支持标准 cron 表达式（5 字段 + 扩展）
- 隔离 session 不共享主 session 历史

## 6. Edge Cases (边界条件)
- Agent 执行超时
- 无 channel recipient 时的 best-effort 投递
- HEARTBEAT_OK 检测（忽略空结果）

## 7. Technical Debt (技术债务)
- 测试文件名过长（describe 内容嵌入文件名）

## 8. Dependencies (依赖)
- **内部**: agents, commands, config, channels
- **外部**: 无特殊外部依赖
