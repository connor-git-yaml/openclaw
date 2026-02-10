# Exec Module Specification

## Intent
The Exec module provides secure process execution capabilities for OpenClaw agents, enabling command execution with proper sandboxing, approval workflows, and session management. It handles background process management, PTY support for interactive commands, and cross-platform process control. The module ensures security through execution policies, timeout handling, and proper cleanup of child processes.

## Interface

### Core Types
```typescript
export type ExecRequest = {
  command: string;
  workdir?: string;
  env?: Record<string, string>;
  timeout?: number;
  pty?: boolean;
  background?: boolean;
  yieldMs?: number;
  elevated?: boolean;
  security?: 'deny' | 'allowlist' | 'full';
  ask?: 'off' | 'on-miss' | 'always';
  host?: 'sandbox' | 'gateway' | 'node';
  node?: string;
};

export type ExecResult = {
  sessionId?: string;
  stdout?: string;
  stderr?: string;
  exitCode?: number;
  killed?: boolean;
  timedOut?: boolean;
  error?: string;
};

export type ProcessSession = {
  id: string;
  command: string;
  pid?: number;
  status: 'running' | 'completed' | 'killed' | 'error';
  startTime: Date;
  endTime?: Date;
  pty?: boolean;
  background: boolean;
};
```

### Key Functions
- `exec(request)` - Execute command with security checks
- `process(action, sessionId?, options?)` - Manage running processes
- `killProcess(sessionId)` - Terminate running process
- `getProcessLogs(sessionId, offset?, limit?)` - Retrieve process output
- `writeToProcess(sessionId, data)` - Send input to running process
- `listActiveSessions()` - Get all active process sessions

### Process Actions
- `list` - List all active process sessions
- `poll` - Get status of specific session
- `log` - Retrieve process output logs
- `write` - Send data to process stdin
- `send-keys` - Send key sequences to PTY
- `paste` - Send clipboard content to process
- `kill` - Terminate process session

## Business Logic

### Command Execution Flow
1. **Security Check**: Validate command against security policies
2. **Environment Setup**: Prepare working directory and environment variables
3. **Process Creation**: Spawn child process with proper configuration
4. **Session Registration**: Register process session for management
5. **Output Monitoring**: Capture stdout/stderr streams
6. **Timeout Handling**: Monitor execution time and apply limits
7. **Cleanup**: Ensure proper resource cleanup on completion

### Security Policy Enforcement
- **Command Allowlisting**: Only approved commands can be executed
- **Directory Restrictions**: Limit access to specific directories
- **Environment Sanitization**: Filter dangerous environment variables
- **Elevated Permission Control**: Require approval for privileged operations
- **Resource Limits**: CPU, memory, and disk usage constraints

### Background Process Management
- **Session Tracking**: Maintain registry of active background processes
- **Automatic Cleanup**: Terminate orphaned processes on restart
- **Resource Monitoring**: Track resource usage of long-running processes
- **Log Rotation**: Prevent unbounded log growth from background processes

### Cross-Platform Support
- **Shell Detection**: Auto-detect appropriate shell (bash/zsh/cmd/powershell)
- **PTY Handling**: Platform-specific pseudo-terminal implementation
- **Path Normalization**: Handle path separators and drive letters
- **Signal Handling**: Cross-platform process termination

## Data Structures

### Process Registry
```typescript
type ProcessRegistry = {
  sessions: Map<string, ProcessSession>;
  cleanup: Map<string, () => void>;
  timeouts: Map<string, NodeJS.Timeout>;
  outputs: Map<string, CircularBuffer<LogEntry>>;
};
```

### Security Context
```typescript
type ExecSecurityContext = {
  allowedCommands: string[];
  blockedCommands: string[];
  workdirRestrictions: string[];
  maxTimeout: number;
  requireApproval: boolean;
  elevatedAllowed: boolean;
};
```

### Log Entry Structure
```typescript
type LogEntry = {
  timestamp: Date;
  stream: 'stdout' | 'stderr' | 'system';
  data: string;
  sessionId: string;
};
```

## Constraints

### Security Constraints
- **Command Validation**: All commands must pass security policy checks
- **Resource Limits**: Maximum 100 concurrent background processes
- **Timeout Enforcement**: Default 30s timeout, maximum 3600s
- **Working Directory**: Must be within allowed directory tree
- **Environment Variables**: Sensitive variables are filtered

### Platform Limitations
- **Windows PTY**: Limited PTY support on Windows systems
- **Process Signals**: Signal handling varies across platforms  
- **File Permissions**: Unix/Windows permission models differ
- **Shell Capabilities**: Feature availability depends on shell type

### Performance Constraints
- **Memory Usage**: Process output buffering limited to 10MB per session
- **CPU Throttling**: Resource limits applied to prevent system overload
- **Concurrent Execution**: Maximum concurrent processes enforced
- **Log Storage**: Process logs have configurable retention limits

## Edge Cases

### Process Termination Scenarios
- **Orphaned Processes**: Parent death before child completion
- **Zombie Processes**: Dead processes not properly reaped
- **Signal Handling**: Graceful vs forceful termination
- **TTY Cleanup**: Proper terminal state restoration

### Error Recovery Patterns
- **Command Not Found**: Graceful handling of missing executables
- **Permission Denied**: Clear error messages for access issues
- **Timeout Recovery**: Cleanup after process timeout expiration
- **Network Interruption**: Handling of remote execution failures

### Concurrency Issues
- **Session ID Conflicts**: Unique session identifier generation
- **Resource Contention**: Multiple processes accessing same resources
- **Output Interleaving**: Proper stream separation and buffering
- **State Synchronization**: Consistent process state across operations

## Technical Debt

### Known Issues
- **Windows Shell Integration**: Inconsistent behavior across Windows shells
- **Process Tree Tracking**: Child process cleanup not comprehensive
- **Resource Monitoring**: Limited real-time resource usage tracking
- **Error Reporting**: Inconsistent error message formatting

### Performance Optimizations Needed
- **Stream Processing**: Large output handling could be optimized
- **Session Storage**: In-memory process registry should persist state
- **Log Compression**: Process logs consume excessive storage
- **Background Cleanup**: More efficient orphaned process detection

### Security Improvements Required
- **Sandbox Integration**: Better container/VM isolation needed
- **Audit Logging**: More comprehensive execution audit trail
- **Command Analysis**: Static analysis of command safety
- **Privilege Escalation**: Better detection of privilege escalation attempts

## Dependencies

### Internal Dependencies
- **Security Module**: Policy enforcement and approval workflows
- **Gateway**: WebSocket communication for process management
- **Logging**: Structured logging of execution events
- **Config**: Security policy and resource limit configuration

### External Dependencies
- **Node.js child_process**: Core process spawning functionality
- **node-pty**: Cross-platform PTY support for interactive commands
- **Signal handling libraries**: Platform-specific signal management
- **Stream processing**: Efficient handling of large process output

### Platform Dependencies
- **Unix/Linux**: bash, zsh, process signals, file permissions
- **Windows**: cmd.exe, PowerShell, Windows process management
- **macOS**: Additional sandboxing and permission handling
- **Container Runtime**: Docker/Podman for enhanced sandboxing