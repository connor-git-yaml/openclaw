---
type: module-spec
version: 1.0
source: src/hooks/
files_analyzed: 22
loc: 3914
last_updated: 2026-02-09
---

# Hooks Spec

## 1. Intent (意图)
Hook 系统模块，支持事件驱动的自动化：
- Gmail 集成（邮件监听和操作）
- Google Calendar 集成
- Hook 配置解析和条件评估
- Frontmatter 解析（hook 定义文件）
- Bundled hooks 目录管理

## 2. Interface (接口定义)
- `resolveHookConfig(config, hookKey)` — 解析 hook 配置
- `isConfigPathTruthy(config, path)` — 检查配置路径是否为真值
- `resolveHookKey(frontmatter)` — 从 frontmatter 解析 hook key
- Gmail ops: 邮件读取、发送、标记等操作
- Gmail watcher: 监听新邮件触发 hook

## 3. Business Logic (业务逻辑)
- **Hook 触发**: 事件发生 → 检查 hook 条件 → 匹配 hook 配置 → 执行 Agent 调用
- **Gmail 监听**: OAuth 认证 → 轮询/push 通知 → 解析邮件 → 触发 hook
- **条件评估**: 基于配置路径的 truthy 检查 + 默认值

## 4. Data Structures (数据结构)
- `HookEntry` — hook 定义
- `HookEligibilityContext` — hook 资格上下文
- `HookConfig` — hook 配置

## 5. Constraints (约束)
- Gmail 需要 Google OAuth 2.0 授权
- Hook 定义通过 markdown frontmatter

## 6. Edge Cases (边界条件)
- OAuth token 过期刷新
- Gmail API 限流
- Hook 条件评估中 undefined 值的处理

## 7. Technical Debt (技术债务)
- Gmail 特定代码占比过高，应考虑插件化

## 8. Dependencies (依赖)
- **内部**: config, agents, commands
- **外部**: Google APIs（gmail, calendar）
