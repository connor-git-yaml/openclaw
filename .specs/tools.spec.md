# Tools Module Specification

## Intent
The Tools module provides the capability system for OpenClaw agents, managing tool registration, execution, security policies, and result processing. It enables agents to interact with external systems, execute commands, manipulate files, search the web, and perform various specialized tasks. The module ensures secure tool execution through sandboxing, policy enforcement, and approval workflows while providing extensibility through plugins and custom tools.

## Interface

### Core Types
```typescript
export type ToolsConfig = {
  profile?: ToolProfileId;
  allow?: string[];
  alsoAllow?: string[];
  deny?: string[];
  byProvider?: Record<string, ToolPolicyConfig>;
  web?: WebToolsConfig;
  media?: MediaToolsConfig;
  message?: MessageToolConfig;
  elevated?: ElevatedToolConfig;
  exec?: ExecToolConfig;
  subagents?: SubagentToolConfig;
  sandbox?: SandboxToolConfig;
}

export type ToolProfileId = "minimal" | "coding" | "messaging" | "full";

export type ToolPolicyConfig = {
  allow?: string[];
  alsoAllow?: string[];
  deny?: string[];
  profile?: ToolProfileId;
}
```

### Key Functions
- `registerTool(definition)` - Register new tool implementation
- `executeTool(toolName, parameters, context)` - Execute tool with security checks
- `validateToolPolicy(toolName, agent, session)` - Check tool access permissions
- `buildToolRegistry(config)` - Create filtered tool registry for agent
- `processToolResult(result, context)` - Handle tool execution results
- `resolveToolDependencies(toolName)` - Get tool dependency chain

### Tool Execution Flow
- **Policy Check**: Verify tool is allowed for current context
- **Parameter Validation**: Validate tool parameters against schema
- **Security Sandbox**: Execute tool in appropriate sandbox environment
- **Result Processing**: Parse and validate tool output
- **Error Handling**: Graceful handling of tool execution failures

## Business Logic

### Tool Registration Process
1. **Discovery**: Scan plugins and built-in modules for tool definitions
2. **Schema Validation**: Verify tool definitions against expected schema
3. **Dependency Resolution**: Build tool dependency graph
4. **Policy Application**: Filter tools based on security policies
5. **Sandbox Configuration**: Configure execution environment for each tool
6. **Registry Creation**: Create per-agent tool registries

### Policy Enforcement System
- **Profile-Based**: Predefined tool sets (minimal, coding, messaging, full)
- **Allow/Deny Lists**: Explicit tool inclusion and exclusion
- **Provider-Specific**: Different tool sets per AI model provider
- **Context-Aware**: Tool availability based on channel/session context
- **Hierarchical**: Agent-level overrides of global policies

### Execution Environment Management
- **Sandbox Routing**: Execute tools in container vs host vs remote node
- **Resource Limits**: CPU, memory, and disk usage constraints
- **Timeout Handling**: Automatic termination of long-running tools
- **Approval Gates**: Human approval required for elevated operations
- **Audit Logging**: Comprehensive logging of tool execution

### Built-in Tool Categories
- **File Operations**: read, write, edit file system operations
- **Command Execution**: exec, process for system command execution
- **Web Tools**: web_search, web_fetch for internet access
- **Communication**: message tool for cross-channel messaging
- **Browser Control**: browser automation and web interaction
- **Media Processing**: image, audio, video analysis tools
- **Agent Management**: sessions, subagent spawning and management

## Data Structures

### Tool Registry
```typescript
interface ToolRegistry {
  tools: Map<string, ToolDefinition>;
  policies: Map<string, ToolPolicy>;
  profiles: Map<ToolProfileId, string[]>;
  dependencies: Map<string, string[]>;
  executors: Map<string, ToolExecutor>;
}
```

### Tool Definition
```typescript
interface ToolDefinition {
  name: string;
  description: string;
  parameters: ToolParameterSchema;
  executor: ToolExecutor;
  category: ToolCategory;
  securityLevel: SecurityLevel;
  dependencies?: string[];
  sandbox?: SandboxRequirement;
  timeout?: number;
}
```

### Tool Execution Context
```typescript
interface ToolExecutionContext {
  agentId: string;
  sessionKey: string;
  toolName: string;
  parameters: Record<string, any>;
  sandbox: SandboxContext;
  permissions: ToolPermissions;
  timeout: number;
  approval?: ApprovalContext;
}
```

### Tool Result
```typescript
interface ToolResult {
  success: boolean;
  data?: any;
  error?: string;
  metadata?: ToolMetadata;
  artifacts?: Artifact[];
  usage?: ResourceUsage;
  duration: number;
}
```

## Constraints

### Security Boundaries
- **Sandbox Isolation**: Tools cannot access host system directly
- **File System Access**: Limited to workspace and temp directories
- **Network Access**: Controlled outbound connectivity
- **Process Execution**: Restricted command execution with allowlists
- **Resource Limits**: CPU, memory, disk, and network quotas

### Performance Limits
- **Execution Timeout**: Default 30 seconds, configurable per tool
- **Concurrent Execution**: Maximum 10 tools per agent simultaneously
- **Memory Usage**: 512MB per tool execution
- **Output Size**: 10MB maximum tool output
- **Queue Length**: 100 pending tool executions per agent

### Tool Complexity
- **Parameter Count**: Maximum 20 parameters per tool
- **Nesting Depth**: Maximum 5 levels of nested parameters
- **Array Size**: Maximum 1000 items in array parameters
- **String Length**: Maximum 100KB per string parameter
- **File Size**: Maximum 100MB for file inputs

### Platform Restrictions
- **Host Dependencies**: Tools requiring specific binaries or libraries
- **Privilege Requirements**: Tools needing elevated system access
- **Platform Compatibility**: OS-specific tool availability
- **Version Dependencies**: Tool compatibility with system versions
- **License Restrictions**: Commercial tool usage limitations

## Edge Cases

### Tool Execution Failures
- **Timeout Exceeded**: Automatic termination and cleanup
- **Resource Exhaustion**: Out of memory or disk space handling
- **Permission Denied**: Graceful fallback for access violations
- **Tool Not Found**: Clear error messages for missing tools
- **Parameter Validation**: Detailed validation error reporting

### Sandbox Complications
- **Container Startup Failure**: Fallback to host execution when allowed
- **Volume Mount Issues**: Workspace access permission problems
- **Network Isolation**: Container networking configuration failures
- **Resource Limit Enforcement**: Proper cleanup when limits exceeded
- **Image Availability**: Handling of missing container images

### Policy Conflicts
- **Conflicting Rules**: Allow vs deny list precedence
- **Provider Overrides**: Model-specific tool restrictions
- **Context Changes**: Tool availability changes during session
- **Permission Updates**: Dynamic permission changes
- **Inheritance Issues**: Policy inheritance in subagent scenarios

### Approval Workflow Issues
- **Approver Unavailable**: Timeout handling for approval requests
- **Concurrent Approvals**: Multiple approval requests management
- **Approval History**: Tracking of approved/denied requests
- **Policy Bypass**: Handling of emergency override scenarios
- **Notification Failures**: Fallback when approval notifications fail

## Technical Debt

### Current Issues
- **Tool Registration**: Manual tool discovery and registration process
- **Parameter Validation**: Inconsistent validation across tools
- **Error Handling**: Non-standardized error response formats
- **Resource Tracking**: Limited resource usage monitoring
- **Security Auditing**: Insufficient audit trail for tool execution

### Architecture Problems
- **Synchronous Execution**: Blocking tool execution affects performance
- **Global Registry**: Shared mutable state across tool instances
- **Tight Coupling**: Direct dependencies between tools and core modules
- **Configuration Sprawl**: Tool settings scattered across multiple files
- **Testing Complexity**: Difficult to test tool execution edge cases

### Performance Bottlenecks
- **Container Startup**: Slow sandbox initialization for each execution
- **File I/O**: Inefficient file operations in sandbox environment
- **Network Calls**: No connection pooling for external API calls
- **JSON Serialization**: Expensive serialization for large tool results
- **Memory Leaks**: Potential memory issues with long-running tools

### Missing Features
- **Tool Composition**: No support for tool chaining or workflows
- **Hot Reload**: Cannot update tool definitions without restart
- **Caching**: No caching of tool results for expensive operations
- **Streaming**: No support for streaming tool outputs
- **Distributed Execution**: No support for multi-node tool execution

## Dependencies

### Core Dependencies
- **jsonschema**: Tool parameter validation
- **docker**: Container-based sandbox execution
- **vm2**: JavaScript code sandbox execution
- **child_process**: System command execution
- **fs**: File system operations

### Security Dependencies
- **chroot**: File system isolation on Unix systems
- **seccomp**: System call filtering for sandboxed processes
- **cgroups**: Resource limit enforcement
- **selinux**: Mandatory access controls
- **apparmor**: Application security profiles

### Tool-Specific Dependencies
- **axios**: Web requests for web_search and web_fetch tools
- **playwright**: Browser automation for browser tool
- **sharp**: Image processing for media tools
- **ffmpeg**: Video/audio processing
- **pdf-parse**: PDF document processing

### Internal Dependencies
- **config**: Tool configuration and policy management
- **agents**: Agent context and session management
- **sandbox**: Sandbox environment configuration
- **auth**: Tool authorization and permissions
- **logging**: Tool execution logging and auditing

### Optional Dependencies
- **kubernetes**: Distributed tool execution on K8s
- **redis**: Tool result caching and job queuing
- **prometheus**: Tool execution metrics
- **opentelemetry**: Distributed tracing for tool calls
- **vault**: Secure credential storage for tool authentication

### Platform Dependencies
- **Node.js**: v18+ for modern JavaScript features and security
- **Docker Engine**: Container runtime for sandboxed execution
- **Linux/Unix**: Full feature support (Windows support limited)
- **Network Access**: Internet connectivity for web-based tools
- **File System**: Workspace and temporary file storage