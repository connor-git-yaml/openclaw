---
type: module-spec
version: 1.0
source: src/infra/
files_analyzed: 121
loc: 23271
last_updated: 2026-02-09
---

# Infra Spec

## 1. Intent (意图)
基础设施模块，提供跨模块共享的底层服务：
- Agent 事件系统（lifecycle, tool, assistant, error 事件流）
- Heartbeat 运行器
- 设备发现（Bonjour/mDNS）
- TLS 证书管理
- 二进制文件管理（外部工具安装）
- 端口管理
- 机器名称/设备身份
- Tailscale/tailnet 集成
- 远程 Skills 注册
- 更新检查
- 归档/压缩
- Backoff 重试策略
- 控制面板 UI 资源
- 诊断事件
- Exec 审批转发

## 2. Interface (接口定义)
- `emitAgentEvent(event)` — 发射 Agent 事件
- `onAgentEvent(listener)` — 监听 Agent 事件
- `registerAgentRunContext(runId, context)` — 注册运行上下文
- `startHeartbeatRunner(cfg)` — 启动心跳运行器
- `loadOrCreateDeviceIdentity()` — 加载/创建设备身份
- `startGatewayDiscovery()` — 启动网关发现服务
- `ensureOpenClawCliOnPath()` — 确保 CLI 在 PATH 中

## 3. Business Logic (业务逻辑)
- **事件系统**: 发布-订阅模式，每个 runId 维护单调递增序号
- **Heartbeat**: 定时触发 → 执行 Agent 调用 → 检查状态
- **设备发现**: Bonjour/mDNS 广播 → 发现对等节点 → 配对
- **TLS**: 自动生成自签名证书 → fingerprint pinning

## 4. Data Structures (数据结构)
- `AgentEventPayload` — Agent 事件（runId, seq, stream, ts, data）
- `AgentRunContext` — 运行上下文（sessionKey, verboseLevel, isHeartbeat）

## 5. Constraints (约束)
- Bonjour 需要 mDNS 网络支持
- TLS 证书存储在 state 目录

## 6. Edge Cases (边界条件)
- mDNS 不可用时的降级
- 证书过期重新生成

## 7. Technical Debt (技术债务)
- 121 个文件，模块过于庞杂，应拆分为多个子模块
- "infra" 是过于宽泛的命名

## 8. Dependencies (依赖)
- **内部**: config, agents, logging
- **外部**: bonjour/ciao（mDNS）, node:tls
