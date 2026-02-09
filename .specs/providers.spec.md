---
type: module-spec
version: 1.0
source: src/providers/
files_analyzed: 4
loc: 411
last_updated: 2026-02-09
---

# Providers Spec

## 1. Intent (意图)
特定 LLM 提供商的认证和模型发现辅助模块：
- GitHub Copilot OAuth 认证和 token 管理
- GitHub Copilot 模型列表发现
- Google 共享工具函数
- Qwen Portal OAuth 认证

## 2. Interface (接口定义)
- `requestGitHubCopilotToken()` — 获取 Copilot 访问 token
- `fetchGitHubCopilotModels()` — 获取可用模型列表
- `qwenPortalOAuth()` — Qwen 门户 OAuth 流程

## 3. Business Logic (业务逻辑)
- **GitHub Copilot**: Device code OAuth 流程 → 轮询获取 access token → 缓存
- **Qwen**: OAuth 2.0 授权码流程

## 4. Data Structures (数据结构)
- `DeviceCodeResponse` — GitHub device code 响应
- `DeviceTokenResponse` — GitHub token 响应

## 5. Constraints (约束)
- 依赖外部 OAuth 端点可用性
- GitHub Copilot 需要有效的 GitHub 账号和 Copilot 订阅

## 6. Edge Cases (边界条件)
- OAuth 超时和轮询间隔处理
- Token 刷新失败的降级

## 7. Technical Debt (技术债务)
- 模块很小，大部分 provider 逻辑实际在 agents 模块中

## 8. Dependencies (依赖)
- **内部**: agents/auth-profiles, commands, config
- **外部**: @clack/prompts
