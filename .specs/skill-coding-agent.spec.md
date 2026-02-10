# Coding Agent Skill Specification

## Intent

Orchestrate AI-powered coding agents (Codex CLI, Claude Code, OpenCode, Pi) through background processes with PTY support. Enables automated code generation, debugging, refactoring, PR reviews, and parallel development workflows for complex software projects.

## Interface

### Supported Agents
- **Codex CLI** - Primary agent using `gpt-5.2-codex` with sandbox modes
- **Claude Code** - Anthropic's coding interface
- **OpenCode** - Alternative coding agent
- **Pi Coding Agent** - Multi-provider coding assistant

### Execution Modes
- **One-shot** - Execute single tasks with immediate completion
- **Background** - Long-running tasks with session management
- **Parallel** - Multiple agents working on different issues simultaneously

### Core Parameters
- `pty: true` - **Required** pseudo-terminal for interactive agents
- `workdir` - Isolated working directory for focused context
- `background` - Session-based execution for monitoring
- `--full-auto` - Auto-approve changes in sandboxed environment
- `--yolo` - No sandbox, no approvals (maximum speed/risk)

## Business Logic

### Agent Orchestration Workflow
1. Validate agent availability and project requirements
2. Establish isolated working directory with git context
3. Launch agent with PTY support and appropriate flags
4. Monitor session progress with background process management
5. Handle agent interactions and user input forwarding
6. Coordinate completion notifications and cleanup

### Session Management
1. Create background sessions for long-running tasks
2. Provide session monitoring and logging capabilities
3. Handle input submission and interactive prompts
4. Manage session termination and resource cleanup
5. Support concurrent sessions for parallel workflows

### Git Integration Patterns
1. Create temporary git repositories for scratch work
2. Use git worktrees for parallel issue resolution
3. Handle PR review workflows with safe isolation
4. Manage branch operations and commit strategies
5. Coordinate push operations and PR creation

### Safety and Isolation
1. Prevent agents from accessing sensitive workspace files
2. Isolate agent execution to specific project directories
3. Validate git repository requirements before execution
4. Handle workspace contamination and cleanup procedures

## Data Structures

### Agent Session
```typescript
interface CodingSession {
  sessionId: string;           // background process identifier
  agent: 'codex' | 'claude' | 'opencode' | 'pi';
  workdir: string;             // isolated working directory
  command: string;             // full command executed
  status: 'running' | 'completed' | 'failed' | 'killed';
  startTime: Date;
  endTime?: Date;
  hasPty: boolean;             // PTY allocation status
  isBackground: boolean;       // background execution flag
}
```

### Agent Configuration
```typescript
interface AgentConfig {
  executable: string;          // binary name (codex, claude, etc.)
  defaultModel?: string;       // model specification
  sandboxMode: 'full-auto' | 'yolo' | 'interactive';
  workspaceIsolation: boolean; // enforce working directory limits
  autoNotification: boolean;   // enable completion wake events
  timeoutMinutes?: number;     // session timeout limit
}
```

### Git Context
```typescript
interface GitWorkspace {
  path: string;                // workspace directory path
  isTemporary: boolean;        // ephemeral workspace flag
  worktreeSource?: string;     // source repository for worktrees
  branchName?: string;         // working branch
  remoteUrl?: string;          // git remote repository
  needsCleanup: boolean;       // requires cleanup after completion
}
```

### Task Progress
```typescript
interface TaskProgress {
  sessionId: string;
  phase: string;               // current task phase
  milestone?: string;          // completed milestone
  needsInput: boolean;         // awaiting user input
  errorMessage?: string;       // failure description
  artifacts: string[];         // generated files/outputs
  estimatedCompletion?: Date;  // projected completion time
}
```

## Constraints

### PTY Requirements
- **Interactive Terminal**: All coding agents require pseudo-terminal allocation
- **Terminal Emulation**: Proper ANSI escape sequence handling for colors/formatting
- **Input/Output**: Bidirectional communication for interactive prompts
- **Session Integrity**: PTY must remain active throughout agent execution

### Git Repository Dependencies
- **Working Directory**: Codex requires git repository context for execution
- **Repository Safety**: Cannot execute in OpenClaw's own repository
- **Branch Isolation**: PR reviews must use separate worktrees or clones
- **Commit Requirements**: Agents may need commit permissions and user configuration

### Resource Management
- **Memory Constraints**: Concurrent agents consume significant system memory
- **CPU Utilization**: Model inference and code generation require processing power
- **Network Bandwidth**: Model API calls require stable internet connectivity
- **Disk Space**: Git worktrees and temporary repositories consume storage

### Security Boundaries
- **Workspace Isolation**: Agents must not access sensitive configuration files
- **Execution Limits**: Sandbox modes provide varying levels of restriction
- **Network Access**: Some agents may require external API connectivity
- **File System**: Write permissions limited to designated working directories

## Edge Cases

### Agent Execution Failures
- **PTY Allocation Failure**: System unable to create pseudo-terminal
- **Agent Binary Missing**: Required coding agent not installed or accessible
- **Git Repository Issues**: Invalid or corrupted git state in working directory
- **Model API Failures**: External AI services unavailable or rate-limited
- **Session Timeout**: Long-running tasks exceeding configured timeout limits

### Interactive Session Problems
- **Input Deadlock**: Agent waiting for input with no user response
- **Output Buffer Overflow**: Excessive agent output causing memory issues
- **Session Abandonment**: Background sessions left running without monitoring
- **Concurrent Input**: Multiple processes attempting to write to same agent
- **Terminal Size Issues**: Agent output formatting problems with terminal dimensions

### Git Workflow Complications
- **Worktree Creation Failure**: Unable to create git worktrees for parallel work
- **Branch Conflicts**: Working on branches that conflict with existing work
- **Remote Push Failures**: Network issues or permission problems pushing changes
- **Merge Conflicts**: Agent-generated changes conflicting with upstream
- **Repository Corruption**: Git state corruption during agent operations

### Resource Exhaustion
- **Memory Leaks**: Long-running agent sessions consuming increasing memory
- **File Descriptor Limits**: Too many concurrent PTY sessions
- **Disk Space Depletion**: Temporary repositories and build artifacts filling storage
- **Network Quota**: API usage limits exceeded for model providers
- **CPU Starvation**: Too many concurrent agents degrading system performance

## Technical Debt

### Process Management Complexity
- Manual session lifecycle management across different agent types
- No unified interface for agent capability detection and configuration
- Limited error recovery and automatic retry mechanisms
- Complex PTY handling with platform-specific edge cases

### Monitoring and Observability Gaps
- Basic session logging without structured event tracking
- No performance metrics for agent execution efficiency
- Limited visibility into agent decision-making processes
- Insufficient debugging tools for failed agent interactions

### Safety and Security Concerns
- Workspace isolation relies on directory-based restrictions
- No comprehensive audit trail for agent-generated changes
- Limited validation of agent-generated code before execution
- Insufficient protection against agents accessing configuration files

### User Experience Limitations
- Command-line interface requires technical expertise
- No graphical progress indicators for long-running tasks
- Limited guidance for optimal agent selection and configuration
- Poor error messages for complex failure scenarios

## Dependencies

### Core Dependencies
- **Bash Execution Environment** - Shell environment for agent command execution
- **Process Management** - Background session handling and PTY allocation
- **Git** - Version control system for workspace management
- **Terminal Emulator** - PTY support for interactive agent communication

### Agent Binaries
- **Codex CLI** - Primary coding agent with GPT-5.2-codex integration
- **Claude Code** - Anthropic's coding interface
- **OpenCode** - Alternative coding agent implementation
- **Pi Coding Agent** - Multi-provider coding assistant (npm package)

### System Dependencies
- **PTY/Terminal Support** - Pseudo-terminal allocation on host system
- **File System** - Read/write access to working directories
- **Network Stack** - Internet connectivity for model API access
- **Memory Management** - Sufficient RAM for concurrent agent execution

### Git Ecosystem
- **Git Worktrees** - For parallel development workflow isolation
- **GitHub CLI** (gh) - For PR management and repository operations
- **Git Hooks** - For automated workflows and validation
- **SSH/HTTPS** - For git remote operations and authentication

### Optional Dependencies
- **Notification System** - Wake events for completion notifications
- **CI/CD Integration** - Automated testing and deployment pipelines
- **Code Analysis Tools** - Static analysis and quality checking
- **Development Environments** - IDE integration and debugging support

### Model Provider APIs
- **OpenAI** - GPT models for Codex and other agents
- **Anthropic** - Claude models with prompt caching support
- **Alternative Providers** - Multi-provider support for Pi agent
- **API Authentication** - Token management for external services