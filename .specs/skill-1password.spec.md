# 1Password Skill Specification

## Intent

Enable secure credential management through 1Password CLI integration. Supports authentication, secret reading, injection, and command execution with secrets for password/API key retrieval without exposing sensitive data in logs or code.

## Interface

### Primary Functions
- **Installation & Setup**: Install 1Password CLI via brew, verify desktop app integration
- **Authentication**: Sign-in flow with desktop app integration, multi-account support
- **Secret Operations**: Read secrets by URI, inject into templates, run commands with secret environment variables
- **Session Management**: tmux-based isolation to maintain authentication state

### Input Methods
- CLI commands through tmux sessions (mandatory isolation)
- Desktop app authentication prompts
- 1Password vault URIs (op://vault/item/field format)

### Output Formats
- Secret values (never logged)
- Authentication status
- Account information
- Error messages with sanitized content

## Business Logic

### Authentication Flow
1. Verify CLI installation and desktop app presence
2. Create isolated tmux session for op commands
3. Execute `op signin` with account specification
4. Handle desktop app authentication prompt
5. Verify success with `op whoami`
6. Maintain session for subsequent operations

### Secret Access Pattern
1. Validate op:// URI format
2. Execute read/inject/run commands within authenticated tmux session
3. Process output without exposing secrets to logs
4. Clean up session after operations

### Multi-Account Support
- Use `--account` flag or `OP_ACCOUNT` environment variable
- Maintain separate authentication sessions per account
- Handle account switching within operations

## Data Structures

### Authentication Session
```typescript
interface OpSession {
  socketPath: string;          // tmux socket location
  sessionName: string;         // unique session identifier
  account: string;             // 1Password account identifier
  authenticated: boolean;      // authentication status
  createdAt: Date;            // session creation time
}
```

### Secret Reference
```typescript
interface SecretRef {
  uri: string;                 // op://vault/item/field format
  vault: string;               // vault name/identifier
  item: string;                // item name/identifier
  field: string;               // field name (password, username, etc.)
  attribute?: string;          // optional attribute (otp, ssh-format)
}
```

### Operation Result
```typescript
interface OpResult {
  success: boolean;
  account: string;
  masked: boolean;             // whether output contains secrets
  error?: string;              // sanitized error message
}
```

## Constraints

### Security Requirements
- **No Secret Logging**: Secrets must never appear in logs, chat, or code
- **tmux Isolation**: All op commands must run in dedicated tmux sessions
- **Desktop Integration**: Requires 1Password desktop app for authentication
- **Session Cleanup**: Temporary sessions must be properly terminated

### Technical Requirements
- **Binary Dependency**: Requires `op` CLI binary (installed via brew)
- **tmux Dependency**: Requires tmux for session isolation
- **Desktop App**: 1Password desktop app must be installed and unlocked
- **Shell Support**: Must work with user's default shell environment

### Platform Limitations
- **macOS Focus**: Optimized for macOS with desktop app integration
- **Network Dependency**: Requires internet connectivity for 1Password service
- **Account Limits**: Bound by 1Password account access permissions

## Edge Cases

### Authentication Failures
- **App Locked**: Desktop app locked during authentication attempt
- **Network Issues**: Connectivity problems during signin
- **Account Suspended**: 1Password account access revoked
- **Token Expiry**: Session expires during long operations

### Session Management
- **Concurrent Sessions**: Multiple op sessions running simultaneously
- **Session Conflicts**: tmux socket naming collisions
- **Orphaned Sessions**: Sessions not properly cleaned up after crashes
- **Permission Issues**: tmux socket permission problems

### Secret Access
- **Invalid URIs**: Malformed op:// references
- **Missing Items**: Referenced vault items not found
- **Permission Denied**: Insufficient access to specific vaults/items
- **Field Ambiguity**: Multiple fields with same name in item

### System State
- **CLI Version Mismatch**: op CLI version incompatible with desktop app
- **Process Conflicts**: Multiple authentication attempts interfering
- **Disk Space**: Insufficient space for tmux socket creation

## Technical Debt

### Session Management Complexity
- Current approach requires manual tmux session lifecycle management
- No automatic session cleanup on unexpected termination
- Socket naming scheme could be improved for better isolation

### Error Handling Gaps
- Insufficient handling of partial authentication states
- Error messages not always sanitized for secret content
- Recovery procedures not well-defined for edge cases

### Testing Limitations
- Difficult to unit test authentication flow without desktop app
- No mock infrastructure for 1Password service interactions
- Integration tests require real 1Password accounts

### Documentation Debt
- Reference examples could be more comprehensive
- Missing troubleshooting guide for common failures
- No performance characteristics documented

## Dependencies

### Direct Dependencies
- **1Password CLI** (`op`): Core binary for all operations
- **tmux**: Session isolation and TTY management
- **1Password Desktop App**: Authentication backend

### System Dependencies
- **Homebrew**: Installation method for op CLI
- **macOS**: Platform assumption for desktop app integration
- **Network Stack**: HTTPS connectivity to 1Password services

### OpenClaw Dependencies
- **tmux Skill**: Session management patterns and socket conventions
- **Shell Tool**: Command execution infrastructure
- **Security Framework**: Credential handling policies

### Optional Dependencies
- **SSH Integration**: For SSH key management use cases
- **Environment File Processing**: For .env file injection workflows
- **Template Systems**: For configuration file injection patterns

### Version Constraints
- **op CLI**: >= 2.0.0 (for modern signin flow)
- **tmux**: >= 3.0 (for socket management features)
- **1Password App**: Latest stable (for reliable desktop integration)