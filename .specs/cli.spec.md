---
type: module-spec
version: 1.0
source: src/cli/
files_analyzed: 138
loc: 21104
last_updated: 2026-02-09
---

# CLI Spec

## 1. Intent (意图)
命令行界面模块，提供 `openclaw` CLI 的参数解析和入口：
- argv 解析和命令路由
- 各子命令的 CLI 参数定义
- 浏览器 CLI 操作（snapshot, screenshot, actions）
- CLI 依赖注入（CliDeps）
- 命令格式化和帮助文本
- ACP（Agent Communication Protocol）CLI

## 2. Interface (接口定义)
- `parseArgv(args)` — 解析命令行参数
- `createDefaultDeps()` → `CliDeps` — 创建默认依赖
- `CliDeps` — CLI 依赖注入接口（config loader, gateway client 等）
- `formatCliCommand(parts)` — 格式化命令显示
- 浏览器 CLI: snapshot, screenshot, actions, debug, extension 管理

## 3. Business Logic (业务逻辑)
- **命令路由**: 解析 argv → 匹配子命令 → 注入 deps → 调用 commands 模块
- **浏览器操作**: 通过 Gateway RPC 代理浏览器命令

## 4. Data Structures (数据结构)
- `CliDeps` — 依赖注入容器
- 各命令的参数类型

## 5. Constraints (约束)
- 需要 Node.js 运行时
- 部分命令需要 Gateway 运行

## 6. Edge Cases (边界条件)
- Gateway 未启动时的提示
- 未知命令的帮助信息

## 7. Technical Debt (技术债务)
- 138 个文件，可按命令分组

## 8. Dependencies (依赖)
- **内部**: commands, config, gateway, browser
- **外部**: yargs 或自定义 argv 解析
