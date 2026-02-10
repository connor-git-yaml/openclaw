---
type: module-spec
version: 1.0
source: src/tui/
files_analyzed: 24
loc: 4155
last_updated: 2026-02-09
---

# TUI Spec

## 1. Intent (意图)
终端用户界面模块，提供交互式聊天体验：
- 与 Gateway 的 WebSocket 实时聊天
- 斜杠命令处理（/model, /session, /clear 等）
- 消息格式化和渲染
- 事件处理（Agent 响应流式显示）

## 2. Interface (接口定义)
- `startTuiChat(opts)` — 启动 TUI 聊天
- TUI 命令处理器
- TUI 事件处理器
- TUI 格式化器

## 3. Business Logic (业务逻辑)
- **聊天循环**: 读取用户输入 → 发送到 Gateway → 流式接收 Agent 响应 → 渲染到终端
- **命令处理**: `/` 前缀 → 解析命令 → 执行（如切换模型、清空 session）

## 4. Data Structures (数据结构)
- TUI 命令定义
- 聊天状态

## 5. Constraints (约束)
- 依赖终端 TTY 支持
- 需要 Gateway 运行

## 6. Edge Cases (边界条件)
- 非 TTY 环境降级
- Gateway 断连重试

## 7. Technical Debt (技术债务)
- 较小模块，职责清晰

## 8. Dependencies (依赖)
- **内部**: gateway/client, config
- **外部**: ink 或自定义终端渲染
