---
type: skill-spec
version: 1.0
source: skills/github/
skill_name: github
files_analyzed: 1
loc: ~100
last_updated: 2026-02-10
---

# GitHub Skill Specification

## 1. Intent (意图)
GitHub Skill 通过 GitHub CLI (`gh`) 提供完整的 GitHub 集成功能：
- 仓库管理和代码浏览
- Issues 和 Pull Request 管理
- CI/CD 工作流监控和操作
- 高级 GitHub API 查询
- 代码审查和协作工作流
- Release 和包管理
- GitHub Actions 集成

该技能作为开发者与 GitHub 生态系统交互的主要界面，支持所有主要的 GitHub 操作。

## 2. Interface (接口定义)
### Core Commands (通过 gh CLI)
- **Issues Management**:
  - `gh issue list/view/create/edit/close` — Issues 管理
  - `gh issue comment/assign/label` — Issues 操作

- **Pull Requests**:
  - `gh pr list/view/create/checkout/merge` — PR 管理
  - `gh pr checks/diff/review` — PR 审查和状态

- **Repository Operations**:
  - `gh repo view/clone/fork/create` — 仓库操作
  - `gh repo set-default` — 默认仓库设置

- **CI/CD Management**:
  - `gh run list/view/rerun/cancel` — 工作流运行管理
  - `gh workflow list/run/view` — 工作流管理

- **Advanced API Access**:
  - `gh api <endpoint>` — 直接 GitHub API 调用
  - JSON 输出和 jq 过滤支持

### Skill Configuration
```yaml
github:
  enabled: true
  default_repo: "owner/repo"  # 可选，默认仓库
  auth_method: "oauth"        # oauth, token, ssh
  output_format: "table"     # table, json, yaml
  pagination_limit: 30       # 分页限制
```

### Authentication Methods
- **OAuth Flow**: 通过 `gh auth login` 进行交互式认证
- **Personal Access Token**: 环境变量或配置文件
- **SSH Key**: Git SSH 密钥认证
- **GitHub App**: 应用程序认证（企业用途）

## 3. Business Logic (业务逻辑)
### 认证和初始化
1. **认证检查**: 验证 `gh auth status` 确认登录状态
2. **CLI安装**: 检查 `gh` CLI 是否安装并为最新版本
3. **权限验证**: 验证当前用户对目标仓库的访问权限
4. **配置加载**: 读取全局和仓库特定的配置

### Issue 和 PR 工作流
1. **列表查询**: 使用过滤条件查询 issues/PRs
2. **详情获取**: 获取具体 issue/PR 的完整信息
3. **状态更新**: 更改状态、添加标签、分配人员
4. **交互操作**: 评论、审查、合并等协作操作

### CI/CD 集成
1. **状态监控**: 实时监控工作流运行状态
2. **失败分析**: 自动分析失败的构建并提供日志
3. **触发操作**: 手动触发工作流或重新运行
4. **结果通知**: 构建状态变更通知

### API 查询优化
1. **查询缓存**: 缓存频繁查询的数据（用户信息、仓库元数据）
2. **批量操作**: 合并多个 API 调用减少延迟
3. **输出格式化**: 使用 jq 过滤和格式化 JSON 输出
4. **错误处理**: GitHub API 错误的友好提示和重试机制

## 4. Data Structures (数据结构)
### Issue 数据结构
```typescript
interface GitHubIssue {
  number: number;
  title: string;
  body: string;
  state: "open" | "closed";
  author: GitHubUser;
  assignees: GitHubUser[];
  labels: GitHubLabel[];
  created_at: string;
  updated_at: string;
  url: string;
}
```

### Pull Request 数据结构
```typescript
interface GitHubPullRequest {
  number: number;
  title: string;
  body: string;
  state: "open" | "closed" | "merged";
  head: GitHubBranch;
  base: GitHubBranch;
  author: GitHubUser;
  reviewers: GitHubUser[];
  checks: GitHubCheck[];
  mergeable: boolean;
  draft: boolean;
}
```

### Workflow Run 数据结构
```typescript
interface GitHubWorkflowRun {
  id: number;
  name: string;
  status: "queued" | "in_progress" | "completed";
  conclusion: "success" | "failure" | "cancelled" | "skipped";
  workflow_id: number;
  head_branch: string;
  head_sha: string;
  created_at: string;
  updated_at: string;
  url: string;
  jobs: GitHubJob[];
}
```

## 5. Constraints (约束)
### GitHub API 限制
- **Rate Limits**: 认证用户每小时5000请求，未认证1000请求
- **Secondary Rate Limits**: 防止滥用的额外限制
- **GraphQL Limits**: GraphQL查询复杂度限制
- **File Size Limits**: 单文件最大100MB，仓库推荐1GB以下

### CLI 工具依赖
- **Version Requirements**: gh CLI >= 2.0.0
- **Platform Support**: macOS, Linux, Windows
- **Network Requirements**: 需要互联网连接访问GitHub
- **Authentication**: 需要有效的GitHub账户和相应权限

### 功能限制
- **Repository Access**: 仅限于用户有权限的仓库
- **Enterprise Features**: 部分功能需要GitHub Enterprise
- **Git Operations**: 某些操作需要本地git环境
- **Large Repositories**: 大型仓库操作可能较慢

## 6. Edge Cases (边界条件)
### 认证问题
- **Token Expiration**: 访问令牌过期的处理和刷新
- **Permission Changes**: 仓库权限变更后的访问失败
- **Multi-Factor Authentication**: 2FA启用时的认证流程
- **Organization Policies**: 组织策略限制的处理

### 网络和API问题
- **GitHub Outages**: GitHub服务中断的降级处理
- **Rate Limit Exceeded**: 触发速率限制时的退避策略
- **Network Timeouts**: 网络超时的重试机制
- **API Changes**: GitHub API变更的兼容性处理

### 仓库状态异常
- **Empty Repositories**: 空仓库的特殊处理
- **Archived Repositories**: 归档仓库的只读操作
- **Private Repositories**: 私有仓库的权限检查
- **Large Files**: Git LFS文件的特殊处理

### CLI 工具问题
- **CLI Not Installed**: gh CLI未安装时的友好提示
- **Version Incompatibility**: 版本不兼容时的升级建议
- **Configuration Conflicts**: CLI配置冲突的解决
- **Command Failures**: CLI命令执行失败的错误处理

## 7. Technical Debt (技术债务)
### 当前问题
- **Limited Configuration**: 配置选项较少，缺乏细粒度控制
- **Error Messages**: CLI错误信息直接传递，缺乏上下文解释
- **Caching Mechanism**: 没有智能缓存减少API调用
- **Batch Operations**: 不支持批量操作，效率较低

### 设计缺陷
- **CLI Dependency**: 完全依赖外部CLI工具，增加故障点
- **Output Parsing**: 依赖CLI输出解析，不够稳定
- **State Management**: 缺乏状态管理，重复查询相同数据
- **Integration Depth**: 与OpenClaw其他模块集成不够深入

### 功能缺失
- **Webhook Support**: 不支持GitHub Webhooks集成
- **Real-time Updates**: 缺乏实时状态更新机制
- **Advanced Analytics**: 缺乏仓库分析和统计功能
- **Team Management**: 团队和权限管理功能有限

### 维护负担
- **CLI Updates**: 需要跟随GitHub CLI版本更新
- **API Changes**: GitHub API变更需要及时适配
- **Feature Parity**: 保持与GitHub web界面功能同步
- **Documentation**: 使用文档需要持续更新

## 8. Dependencies (依赖)
### 外部依赖
- **GitHub CLI**: `gh` 命令行工具 (>= 2.0.0)
- **Git**: 本地git环境（某些操作需要）
- **Network**: 稳定的互联网连接
- **GitHub Account**: 有效的GitHub账户和认证

### 系统依赖
- **Operating System**: macOS, Linux, Windows支持
- **Shell Environment**: 支持命令行执行
- **JSON Processing**: jq工具（可选，增强输出处理）
- **SSH**: SSH客户端（SSH认证时需要）

### OpenClaw内部依赖
- **config**: 技能配置管理
- **logging**: 操作日志记录
- **security**: 认证信息安全存储
- **utils**: 通用工具函数

### 可选增强依赖
- **Visual Tools**: GitHub Desktop（可视化操作）
- **Editor Integration**: VS Code GitHub扩展
- **Notification Services**: GitHub移动应用
- **Analytics Tools**: GitHub Insights工具

---

## 常用操作示例

### Issue管理
```bash
# 列出当前仓库的开放Issues
gh issue list --state open

# 创建新Issue
gh issue create --title "Bug: Login failed" --body "Description of the bug"

# 为Issue添加标签
gh issue edit 123 --add-label "bug,priority-high"
```

### Pull Request操作
```bash
# 检查PR的CI状态
gh pr checks 55 --repo owner/repo

# 审查PR代码
gh pr review 55 --approve --body "LGTM!"

# 合并PR
gh pr merge 55 --merge --delete-branch
```

### CI/CD监控
```bash
# 查看最近的工作流运行
gh run list --limit 10

# 查看失败的构建日志
gh run view 123456 --log-failed

# 重新运行失败的构建
gh run rerun 123456 --failed
```

该技能为开发团队提供了完整的GitHub工作流集成，支持从代码提交到发布的全流程管理。