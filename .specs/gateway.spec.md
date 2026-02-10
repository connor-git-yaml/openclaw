---
type: module-spec
version: 1.0
source: src/gateway/
files_analyzed: 133
loc: 26364
last_updated: 2026-02-09
---

# Gateway Spec

## 1. Intent (意图)
Gateway 是 OpenClaw 的核心服务端，作为 WebSocket/HTTP JSON-RPC 服务器运行。它是所有客户端（CLI、TUI、Discord、Telegram 等）与 AI Agent 之间的中枢，负责：
- 管理 WebSocket 连接和 JSON-RPC 协议通信
- 路由消息到正确的 Agent 和 Session
- 管理 Channel 插件（Discord/Telegram/Slack 等）的生命周期
- 提供 HTTP API（OpenAI 兼容、Open Responses）
- 管理节点注册（Node Registry）和设备配对
- 执行审批（Exec Approval）管理
- Boot 脚本执行、配置热加载、Cron 调度

## 2. Interface (接口定义)
- `startGatewayServer(opts: GatewayServerOptions): Promise<GatewayServer>` — 启动网关服务
- `GatewayClient` — WebSocket 客户端，用于 CLI/TUI 连接网关
- `callGateway(opts: CallGatewayOptions)` — 调用网关 RPC 方法
- `buildGatewayConnectionDetails()` — 解析网关连接参数（URL、TLS、auth）
- JSON-RPC 方法集：`coreGatewayHandlers` 提供 chat、session、config、browser、cron 等方法
- HTTP 端点：OpenAI 兼容 `/v1/chat/completions`、Open Responses API
- `NodeRegistry` — 管理已配对的移动/远程节点
- `ExecApprovalManager` — 管理命令执行审批流

## 3. Business Logic (业务逻辑)
- **启动流程**: 加载配置 → 迁移旧配置 → 初始化 TLS → 启动 WebSocket 服务器 → 加载 Channel 插件 → 启动 Heartbeat/Cron → 启动发现服务 → 执行 Boot 脚本
- **消息路由**: 客户端连接 → 认证（token/password/device-auth）→ 解析 session key → 路由到对应 Agent
- **配置热加载**: 文件变更监听 → 重新加载配置 → 通知已连接客户端
- **节点管理**: Bonjour/mDNS 发现 → 配对审批 → 注册到 NodeRegistry → 转发命令
- **Health 状态**: 定期刷新健康快照，供客户端轮询

## 4. Data Structures (数据结构)
- `GatewayServerOptions` — 服务器配置（端口、TLS、runtime 等）
- `CallGatewayOptions` — 客户端调用参数（url、token、method、params）
- `GatewayConnectionDetails` — 连接详情（url、urlSource、message）
- `ControlUiRootState` — 控制面板 UI 状态
- `PROTOCOL_VERSION` — 协议版本号，用于客户端兼容性检查

## 5. Constraints (约束)
- 依赖 Node.js `ws` 库实现 WebSocket
- 支持 TLS（自签名证书 + fingerprint pinning）
- 支持 Tailscale 网络暴露
- JSON-RPC 2.0 协议
- 需要文件系统访问（配置、session 存储）

## 6. Edge Cases (边界条件)
- 远程模式（remote gateway）vs 本地模式
- TLS 证书自动生成和 fingerprint 管理
- 多客户端并发连接和 session 隔离
- Boot 脚本执行失败的降级处理
- 配置文件损坏时的容错

## 7. Technical Debt (技术债务)
- `server.impl.ts` 文件过大（启动逻辑复杂，导入超过 60 个模块）
- 部分方法散落在多个 `server-*.ts` 文件中，缺乏统一抽象
- Health 状态使用全局缓存，可能有并发问题

## 8. Dependencies (依赖)
- **内部**: agents, channels, config, routing, infra, logging, security, cron, browser, hooks, memory, commands
- **外部**: ws, chokidar（文件监听）
