---
type: module-spec
version: 1.0
source: extensions/
files_analyzed: 32
loc: ~15000 (estimated)
last_updated: 2026-02-10
---

# Extensions System Specification

## 1. Intent (意图)
Extensions System 是 OpenClaw 的核心通讯平台集成系统，专门负责：
- 实时通讯平台连接（Discord、Telegram、Slack等）
- 消息协议适配和标准化
- 身份认证和会话管理
- 跨平台消息路由和转发
- 专用channel插件扩展
- 实时事件监听和处理

与Skills不同，Extensions专注于双向实时通讯集成，作为Agent与外部世界交互的主要通道。

## 2. Interface (接口定义)
### Extension 生命周期管理
- `loadExtension(extensionPath: string)` — 加载扩展模块
- `registerChannelProvider(provider: ChannelProvider)` — 注册通道提供者
- `startExtension(extensionName: string)` — 启动扩展服务
- `stopExtension(extensionName: string)` — 停止扩展服务

### 消息处理接口
- `sendMessage(channelId: string, message: Message)` — 发送消息
- `receiveMessage(callback: MessageHandler)` — 注册消息接收处理器
- `transformMessage(message: RawMessage): NormalizedMessage` — 消息格式转换
- `routeMessage(message: NormalizedMessage)` — 消息路由

### 认证和连接管理
- `authenticate(credentials: AuthCredentials)` — 平台认证
- `establishConnection()` — 建立连接
- `handleReconnection()` — 处理重连
- `validateSession()` — 会话验证

### Extension 类别
- **Major Platforms**: discord, telegram, slack, msteams
- **Chat Platforms**: googlechat, line, matrix, mattermost
- **Mobile Messaging**: imessage, bluebubbles, signal, whatsapp
- **International**: feishu, zalo, zalouser
- **Specialized**: nostr, nextcloud-talk, tlon
- **Enterprise**: msteams, googlechat, slack
- **Voice/Video**: voice-call, twitch
- **Authentication**: google-antigravity-auth, minimax-portal-auth, qwen-portal-auth
- **AI Integration**: llm-task, lobster, open-prose
- **Storage**: memory, memory-core, memory-lancedb
- **Development**: copilot-proxy, diagnostics-otel

## 3. Business Logic (业务逻辑)
### Extension 启动流程
1. **配置加载**: 读取平台特定配置和认证信息
2. **认证验证**: 验证API密钥、tokens或其他认证凭据
3. **连接建立**: 建立WebSocket、HTTP或其他协议连接
4. **事件注册**: 注册消息、状态变更等事件监听器
5. **健康检查**: 启动定期健康检查和监控

### 消息处理流程
1. **原始消息接收**: 从平台接收原始格式消息
2. **消息标准化**: 转换为OpenClaw统一消息格式
3. **路由决策**: 根据规则决定路由目标Agent
4. **Agent处理**: 交给Agent处理并等待响应
5. **响应转换**: 将Agent响应转换为平台特定格式
6. **消息发送**: 发送到目标平台

### 状态管理
- **连接状态**: 在线、离线、重连中、错误
- **认证状态**: 已认证、未认证、过期、无效
- **消息队列**: 处理连接中断时的消息缓存
- **错误恢复**: 自动重试和错误处理机制

## 4. Data Structures (数据结构)
```typescript
interface Extension {
  name: string;
  version: string;
  platform: Platform;
  status: ExtensionStatus;
  connection: Connection;
  config: ExtensionConfig;
}

interface ChannelProvider {
  platform: Platform;
  capabilities: Capability[];
  messageHandler: MessageHandler;
  authHandler: AuthHandler;
  connectionManager: ConnectionManager;
}

interface NormalizedMessage {
  id: string;
  platform: Platform;
  channelId: string;
  authorId: string;
  content: string;
  timestamp: Date;
  attachments?: Attachment[];
  metadata: MessageMetadata;
}

enum ExtensionStatus {
  LOADING = "loading",
  CONNECTED = "connected",
  DISCONNECTED = "disconnected", 
  ERROR = "error",
  RECONNECTING = "reconnecting"
}

interface Platform {
  name: string;
  type: PlatformType;
  features: PlatformFeature[];
  limits: PlatformLimits;
}
```

## 5. Constraints (约束)
### 性能限制
- **Concurrent Connections**: 最大同时连接50个平台
- **Message Throughput**: 每秒处理1000条消息
- **Memory Usage**: 每个extension最大内存使用200MB
- **Response Time**: 消息响应延迟小于2秒

### 平台限制
- **Rate Limits**: 遵守各平台API速率限制
- **Message Size**: 适配各平台消息长度限制
- **File Upload**: 文件上传大小和格式限制
- **Connection Limits**: 各平台并发连接数限制

### 安全约束
- **Credential Encryption**: 认证信息加密存储
- **Access Control**: 基于角色的访问控制
- **Audit Logging**: 详细的操作审计日志
- **Privacy Protection**: 消息内容隐私保护

## 6. Edge Cases (边界条件)
### 连接异常
- **Network Interruption**: 网络中断的重连处理
- **Platform Outages**: 平台服务中断的降级处理
- **Authentication Expiry**: 认证过期的自动刷新
- **Rate Limit Exceeded**: 速率限制触发的退避策略

### 消息处理异常
- **Malformed Messages**: 格式错误消息的处理
- **Large Attachments**: 超大附件的分割传输
- **Encoding Issues**: 字符编码问题的处理
- **Duplicate Messages**: 重复消息的去重处理

### 平台差异
- **Feature Incompatibility**: 不同平台功能差异的适配
- **Message Format Differences**: 消息格式差异的标准化
- **Emoji and Rich Content**: 表情符号和富文本的转换
- **Threading vs Channels**: 不同组织结构的映射

## 7. Technical Debt (技术债务)
### 当前问题
- **Code Duplication**: 多个extension间存在重复代码
- **Error Handling**: 错误处理机制不统一，缺乏标准化
- **Testing Coverage**: 大部分extension缺乏集成测试
- **Documentation**: 部分extension文档不完整或过时
- **Configuration Complexity**: 配置格式复杂，用户体验差

### 架构问题
- **Tight Coupling**: extension与gateway模块耦合过紧
- **State Management**: 缺乏统一的状态管理机制
- **Message Queue**: 没有持久化的消息队列机制
- **Hot Reload**: 不支持extension的热重载

### 维护负担
- **API Changes**: 第三方平台API变更适配成本高
- **Version Management**: 多平台SDK版本管理复杂
- **Dependency Conflicts**: 不同extension依赖版本冲突
- **Security Updates**: 安全补丁更新响应慢

## 8. Dependencies (依赖)
### 核心依赖
- **gateway**: 网关服务和路由
- **channels**: 通道管理系统
- **config**: 配置管理
- **logging**: 日志记录
- **security**: 认证和授权

### 平台SDK依赖
- **discord.js**: Discord平台集成
- **@slack/bolt**: Slack平台集成
- **telegraf**: Telegram Bot API
- **@microsoft/teams-js**: Microsoft Teams集成
- **matrix-js-sdk**: Matrix协议支持

### 外部服务依赖
- **WebSocket Libraries**: ws, socket.io等实时通讯库
- **HTTP Clients**: axios, fetch等HTTP请求库
- **Authentication**: OAuth2、JWT等认证库
- **Encryption**: 消息加密和签名库

---

## 重要Extension模块清单

### 主流平台 (Major Platforms)
- **discord**: Discord服务器和DM集成
- **telegram**: Telegram bot和群组管理
- **slack**: Slack工作区和频道集成
- **msteams**: Microsoft Teams集成

### 移动通讯 (Mobile Messaging)
- **imessage**: iMessage集成 (macOS)
- **bluebubbles**: 跨平台iMessage服务
- **signal**: Signal私密通讯
- **whatsapp**: WhatsApp商业API

### 国际化平台 (International)
- **feishu**: 飞书企业通讯
- **zalo**: Zalo越南即时通讯
- **line**: LINE日韩通讯平台
- **googlechat**: Google Chat企业版

### 专业平台 (Specialized)
- **matrix**: 去中心化通讯协议
- **nostr**: 去中心化社交网络协议
- **tlon**: Urbit生态通讯
- **twitch**: Twitch直播聊天

### AI和开发 (AI & Development)
- **llm-task**: LLM任务处理
- **copilot-proxy**: GitHub Copilot代理
- **lobster**: AI助手集成
- **open-prose**: 开放文本处理

### 认证服务 (Authentication)
- **google-antigravity-auth**: Google服务认证
- **minimax-portal-auth**: MiniMax AI平台认证
- **qwen-portal-auth**: 通义千问平台认证

### 基础设施 (Infrastructure)
- **memory-core**: 核心内存管理
- **memory-lancedb**: 向量数据库集成
- **diagnostics-otel**: OpenTelemetry诊断

每个Extension的详细实现规范见其各自目录下的源码和文档。