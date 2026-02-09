---
type: module-spec
version: 1.0
source: src/config/
files_analyzed: 89
loc: 14329
last_updated: 2026-02-09
---

# Config Spec

## 1. Intent (意图)
OpenClaw 的配置中心，负责：
- 配置文件（JSON5）的读取、写入、解析和验证
- Zod schema 定义所有配置类型
- 配置路径解析（state dir, config path, workspace）
- Session 存储管理
- Agent 限制和命令配置
- 频道能力（channel capabilities）声明
- 遗留配置迁移
- 插件自动启用

## 2. Interface (接口定义)
- `loadConfig(): OpenClawConfig` — 加载并缓存配置
- `writeConfigFile(config)` — 写入配置文件
- `readConfigFileSnapshot()` — 读取原始配置快照
- `parseConfigJson5(text)` — 解析 JSON5 配置
- `resolveConfigPath(env, stateDir)` — 解析配置文件路径
- `resolveStateDir(env)` — 解析状态目录
- `resolveGatewayPort(config)` — 解析网关端口
- `OpenClawConfig` — 核心配置类型（通过 Zod schema 生成）
- `OpenClawSchema` — Zod schema 对象
- `validateConfigObject(obj)` — 验证配置对象
- `migrateLegacyConfig()` — 迁移旧版配置

## 3. Business Logic (业务逻辑)
- **配置加载**: 读取 JSON5 文件 → Zod 验证 → 缓存 → 返回类型安全对象
- **Session 管理**: 基于文件系统的 session 存储 → 按 agent/session key 组织
- **类型分拆**: types.ts 按领域拆分为 30+ 子类型文件（types.agents, types.discord, types.gateway 等）
- **插件自动启用**: 检测已安装的 channel 插件 → 自动添加到配置

## 4. Data Structures (数据结构)
- `OpenClawConfig` — 顶级配置类型，包含 gateway, agents, bindings, channels, hooks, cron, browser, memory, security 等所有子配置
- `AgentBinding` — Agent 到 channel/peer 的绑定规则
- `SessionEntry` — 会话条目（模型覆盖、发送策略等）
- `HookConfig` — Hook 配置
- `CronConfig` — 定时任务配置

## 5. Constraints (约束)
- 配置文件格式为 JSON5（支持注释）
- 配置路径默认 `~/.openclaw/config.json5`
- State 目录默认 `~/.openclaw/`
- Zod schema 提供运行时类型验证

## 6. Edge Cases (边界条件)
- 配置文件不存在时使用默认值
- JSON5 解析失败的错误处理
- 遗留配置格式自动迁移
- Nix 模式下的路径差异

## 7. Technical Debt (技术债务)
- types.ts 拆分为 30+ 文件，虽然每个文件小但导航成本高
- 配置加载使用同步缓存，可能在并发场景有问题

## 8. Dependencies (依赖)
- **内部**: agents, channels, routing
- **外部**: zod, json5
