---
type: module-spec
version: 1.0
source: skills/
files_analyzed: 52
loc: ~25000 (estimated)
last_updated: 2026-02-10
---

# Skills System Specification

## 1. Intent (意图)
Skills System 是 OpenClaw 的可扩展功能模块系统，提供：
- 第三方服务集成（API、CLI工具、设备）
- 专用工具封装（笔记、日历、媒体、通讯等）
- 跨平台设备控制（HomePod、Hue灯、摄像头等）
- AI能力扩展（图像生成、语音合成、文档处理等）
- 工作流自动化（任务管理、文件处理、通知等）

每个Skill都是独立的模块，有自己的配置、依赖和SKILL.md文档。

## 2. Interface (接口定义)
### Skill 发现和加载
- `scanSkills(skillsDir: string)` — 扫描技能目录
- `loadSkill(skillPath: string)` — 加载单个技能
- `validateSkillStructure(skill: Skill)` — 验证技能结构
- `getSkillManifest(skillDir: string)` — 读取技能清单

### Skill 执行接口
- `executeSkillCommand(skillName: string, command: string, args: any[])` — 执行技能命令
- `getSkillStatus(skillName: string)` — 获取技能状态
- `enableSkill(skillName: string)` — 启用技能
- `disableSkill(skillName: string)` — 禁用技能

### 技能类别
- **Communication**: discord, slack, telegram, imsg, bluebubbles, voice-call
- **Note Taking**: apple-notes, bear-notes, notion, obsidian
- **Task Management**: apple-reminders, things-mac, trello
- **Media & Entertainment**: spotify-player, sonoscli, gifgrep, songsee
- **Smart Home**: openhue, camsnap, peekaboo
- **Development**: github, coding-agent, tmux, skill-creator
- **AI Services**: gemini, openai-image-gen, openai-whisper, sag
- **Productivity**: 1password, himalaya, weather, oracle
- **System Integration**: wacli, healthcheck, session-logs, video-frames

## 3. Business Logic (业务逻辑)
### 技能生命周期
1. **发现**: 扫描skills目录，识别有效的技能模块
2. **验证**: 检查SKILL.md、依赖、配置格式
3. **加载**: 读取配置、注册命令、初始化资源
4. **执行**: 响应Agent调用、执行具体功能
5. **监控**: 健康检查、错误报告、性能统计
6. **更新**: 热重载配置、版本更新

### 依赖管理
- **System Dependencies**: CLI工具、系统库、环境变量
- **External APIs**: 第三方服务认证、API密钥管理
- **Internal Dependencies**: OpenClaw内部模块依赖
- **Runtime Requirements**: Node.js版本、平台兼容性

### 配置系统
- **Skill-specific Config**: 每个技能的独立配置段
- **Credential Management**: 安全的API密钥和认证信息存储
- **Environment Variables**: 环境变量注入和替换
- **Default Values**: 合理的默认配置值

## 4. Data Structures (数据结构)
```typescript
interface Skill {
  name: string;
  path: string;
  manifest: SkillManifest;
  config: SkillConfig;
  status: SkillStatus;
  commands: SkillCommand[];
}

interface SkillManifest {
  name: string;
  version: string;
  description: string;
  author?: string;
  dependencies?: string[];
  platforms?: string[];
  capabilities: string[];
}

interface SkillConfig {
  enabled: boolean;
  credentials?: Record<string, any>;
  options?: Record<string, any>;
  environment?: Record<string, string>;
}

enum SkillStatus {
  UNKNOWN = "unknown",
  AVAILABLE = "available", 
  DISABLED = "disabled",
  ERROR = "error",
  MISSING_DEPS = "missing_deps"
}
```

## 5. Constraints (约束)
### 技能数量限制
- **Maximum Skills**: 100个并发活跃技能
- **Memory Usage**: 每个技能最大内存使用500MB
- **Execution Time**: 单次命令执行超时60秒
- **API Rate Limits**: 遵守各第三方服务的速率限制

### 安全约束
- **Sandboxing**: 技能运行在受限环境中
- **Credential Isolation**: 技能间认证信息隔离
- **File System Access**: 限制文件系统访问范围
- **Network Access**: 可配置的网络访问控制

### 兼容性要求
- **Platform Support**: 跨平台兼容性（macOS, Linux, Windows）
- **Node.js Version**: 最低Node.js v16支持
- **Dependency Versions**: 严格的依赖版本管理
- **API Compatibility**: 向后兼容的API接口

## 6. Edge Cases (边界条件)
### 技能失效场景
- **Missing Dependencies**: 依赖工具未安装或版本不兼容
- **API Failures**: 第三方服务不可用或认证失败
- **Configuration Errors**: 配置文件格式错误或缺失必需参数
- **Permission Issues**: 文件系统权限或网络访问被拒绝
- **Resource Exhaustion**: 内存、CPU或网络资源耗尽

### 并发控制
- **Skill Conflicts**: 多个技能访问相同资源的冲突处理
- **Rate Limiting**: 防止技能过度调用外部API
- **Resource Sharing**: 共享资源的并发访问控制
- **Deadlock Prevention**: 避免技能间相互等待造成死锁

### 升级和迁移
- **Skill Updates**: 技能版本升级时的向后兼容性
- **Configuration Migration**: 配置格式变更时的数据迁移
- **Dependency Conflicts**: 不同技能依赖版本冲突解决
- **Rollback Support**: 升级失败时的回滚机制

## 7. Technical Debt (技术债务)
### 当前问题
- **Documentation Inconsistency**: SKILL.md格式不统一，部分文档过时
- **Error Handling**: 错误处理机制不完善，缺乏统一的错误格式
- **Testing Coverage**: 大部分技能缺乏自动化测试
- **Performance Monitoring**: 缺乏技能性能监控和优化机制
- **Dependency Management**: 依赖版本管理混乱，容易产生冲突

### 架构问题
- **Coupling**: 部分技能与OpenClaw核心模块耦合过紧
- **Standardization**: 缺乏统一的技能开发标准和最佳实践
- **Discovery Mechanism**: 技能发现机制简单，缺乏元数据索引
- **Hot Reload**: 技能热重载机制不完善，需要重启服务

### 安全隐患
- **Credential Storage**: 部分技能明文存储敏感信息
- **Input Validation**: 缺乏统一的输入验证和清洗机制
- **Privilege Escalation**: 某些技能可能获得超出必要的系统权限
- **Audit Trail**: 缺乏详细的技能操作审计日志

## 8. Dependencies (依赖)
### 核心依赖
- **config**: 技能配置管理
- **logging**: 日志记录和调试
- **agents**: Agent系统集成
- **gateway**: 网关通信
- **security**: 认证和权限管理

### 外部依赖
- **CLI Tools**: 各种命令行工具（git, curl, ffmpeg等）
- **System Libraries**: 系统级库文件和驱动
- **Third-party APIs**: 外部服务API（Discord, Spotify, OpenAI等）
- **Package Managers**: npm, pip, brew等包管理器

### 特定技能依赖
- **Apple Integration**: macOS系统APIs, AppleScript
- **Smart Home**: Philips Hue API, HomeKit framework  
- **Media Processing**: FFmpeg, ImageMagick
- **Communication**: Discord.js, Slack API, Telegram Bot API
- **AI Services**: OpenAI API, Google AI APIs

---

## 重要技能模块清单

### 通讯类 (Communication)
- **discord**: Discord bot集成和消息管理
- **slack**: Slack工作区集成  
- **telegram**: Telegram bot支持
- **imsg**: iMessage集成(macOS)
- **bluebubbles**: 跨平台iMessage访问
- **voice-call**: 语音通话功能

### 笔记管理 (Note Taking)
- **apple-notes**: Apple Notes集成
- **bear-notes**: Bear笔记应用集成
- **notion**: Notion API集成
- **obsidian**: Obsidian笔记管理

### AI和媒体 (AI & Media)
- **openai-image-gen**: OpenAI图像生成
- **openai-whisper**: 语音转文字
- **sag**: ElevenLabs TTS语音合成
- **gemini**: Google Gemini AI集成

### 智能家居 (Smart Home)  
- **openhue**: Philips Hue智能灯控制
- **camsnap**: 摄像头抓拍
- **peekaboo**: 设备监控

### 开发工具 (Development)
- **github**: GitHub集成
- **coding-agent**: 代码助手
- **tmux**: 终端会话管理
- **skill-creator**: 技能创建向导

完整的52个技能详细信息见各自的SKILL.md文档。