# MCPorter Skill Specification

## Intent

Provide comprehensive Model Context Protocol (MCP) server management and interaction capabilities through CLI interface. Enables MCP server discovery, tool invocation, authentication, configuration management, and code generation for protocol-based AI tool integration.

## Interface

### Core Operations
- **Server Management**: List, configure, and manage MCP servers
- **Tool Invocation**: Call MCP server tools with various parameter formats
- **Authentication**: OAuth and credential management for MCP servers
- **Configuration**: Server configuration import/export and management
- **Code Generation**: CLI and TypeScript client generation from MCP schemas

### Command Structure
- `mcporter list [SERVER] [--schema]` - Enumerate servers and tools
- `mcporter call SERVER.TOOL key=value` - Invoke MCP server tools
- `mcporter auth SERVER [--reset]` - Authentication management
- `mcporter config list|get|add|remove` - Configuration operations
- `mcporter generate-cli --server NAME` - Generate CLI wrappers
- `mcporter daemon start|stop|restart` - Background service management

### Tool Invocation Methods
- **Selector syntax**: `server.tool key=value`
- **Function syntax**: `server.tool(param: "value")`
- **Direct URL**: Full HTTP endpoint with parameters
- **Stdio**: Local server execution with stdio communication
- **JSON payload**: Structured argument passing

### Output Formats
- Human-readable default formatting
- JSON output (`--output json`) for machine processing
- Schema inspection for tool discovery

## Business Logic

### MCP Protocol Implementation
1. Implement Model Context Protocol communication standards
2. Handle HTTP and stdio transport mechanisms for server communication
3. Manage protocol versioning and capability negotiation
4. Process MCP message formatting and error handling

### Server Discovery and Management
1. Maintain registry of available MCP servers and configurations
2. Support ad-hoc server connections and temporary configurations
3. Handle server lifecycle management (start, stop, health checks)
4. Provide server schema inspection and tool enumeration

### Authentication and Authorization
1. Implement OAuth flow for MCP server authentication
2. Manage credential storage and refresh token handling
3. Support multiple authentication methods per server
4. Handle authorization scopes and permission validation

### Tool Invocation Framework
1. Parse various parameter syntax formats for tool calls
2. Validate tool parameters against server schemas
3. Execute tool calls with proper error handling and retries
4. Format and present tool execution results

### Code Generation Pipeline
1. Analyze MCP server schemas for tool definitions
2. Generate CLI wrapper scripts for server tools
3. Create TypeScript client libraries and type definitions
4. Support custom templates and generation customization

## Data Structures

### MCP Server Configuration
```typescript
interface MCPServer {
  name: string;                // server identifier
  url?: string;                // HTTP server endpoint
  command?: string[];          // stdio server command
  transport: 'http' | 'stdio';
  auth?: AuthConfig;           // authentication configuration
  capabilities: string[];      // supported MCP capabilities
  tools: MCPTool[];           // available tools
  lastSeen: Date;             // last successful connection
  isActive: boolean;          // server status
}
```

### MCP Tool Definition
```typescript
interface MCPTool {
  name: string;               // tool identifier
  server: string;             // owning server name
  description: string;        // tool functionality description
  parameters: ParameterSchema; // input parameter schema
  returns?: Schema;           // return value schema
  examples: CallExample[];    // usage examples
  category?: string;          // tool categorization
}
```

### Tool Call Request
```typescript
interface ToolCallRequest {
  server: string;             // target server name
  tool: string;               // tool name
  parameters: Record<string, any>; // tool arguments
  format: 'selector' | 'function' | 'json';
  timeout?: number;           // call timeout seconds
  retries?: number;           // retry attempts
}
```

### Authentication Configuration
```typescript
interface AuthConfig {
  type: 'oauth' | 'apikey' | 'basic' | 'none';
  clientId?: string;          // OAuth client identifier
  clientSecret?: string;      // OAuth client secret
  scope?: string[];           // OAuth permission scopes
  tokenEndpoint?: string;     // OAuth token URL
  refreshToken?: string;      // stored refresh token
  accessToken?: string;       // current access token
  expires?: Date;             // token expiration time
}
```

### Generation Configuration
```typescript
interface GenerationConfig {
  target: 'cli' | 'typescript-client' | 'typescript-types';
  server: string;             // source server name
  outputPath?: string;        // generation destination
  templatePath?: string;      // custom template location
  includeExamples: boolean;   // include usage examples
  namespace?: string;         // TypeScript namespace
  exportFormat: 'esm' | 'cjs'; // module format
}
```

### Daemon Configuration
```typescript
interface DaemonConfig {
  port: number;               // daemon HTTP port
  logLevel: 'debug' | 'info' | 'warn' | 'error';
  maxConcurrentCalls: number; // concurrent tool call limit
  callTimeout: number;        // default call timeout
  serverHealthInterval: number; // server ping frequency
  configReloadInterval: number; // config file watch interval
}
```

## Constraints

### MCP Protocol Limitations
- **Protocol Compliance**: Must adhere to MCP specification requirements
- **Transport Support**: Limited to HTTP and stdio communication methods
- **Schema Validation**: Tool calls must conform to server-defined schemas
- **Version Compatibility**: MCP protocol version mismatches may cause failures

### Server Connectivity Requirements
- **Network Access**: HTTP servers require network connectivity
- **Process Management**: Stdio servers require local execution capabilities
- **Authentication**: OAuth flows require browser access for user consent
- **Resource Limits**: Server connections consume memory and file descriptors

### Code Generation Constraints
- **Schema Availability**: Generation requires complete server schema access
- **Template Dependencies**: Custom templates must match expected format
- **Language Support**: TypeScript generation primary focus
- **Output Validation**: Generated code must compile and execute correctly

### Configuration Management
- **File Permissions**: Configuration files require read/write access
- **JSON Validity**: Configuration must be valid JSON format
- **Credential Security**: Sensitive authentication data needs secure storage
- **Version Migration**: Configuration format changes require migration logic

## Edge Cases

### Server Connectivity Issues
- **HTTP Server Offline**: Target servers unavailable or unresponsive
- **Stdio Server Failures**: Local server commands failing to start or crashing
- **Network Timeouts**: Slow networks causing tool call failures
- **Authentication Expiry**: OAuth tokens expiring during operations
- **Protocol Version Mismatch**: Server and client MCP version conflicts

### Tool Call Complications
- **Parameter Validation Errors**: Invalid tool arguments causing failures
- **Tool Not Found**: Requesting non-existent tools from servers
- **Response Format Issues**: Server responses not matching expected schema
- **Concurrent Call Limits**: Too many simultaneous tool calls causing throttling
- **Large Response Handling**: Oversized tool responses causing memory issues

### Configuration and Management Problems
- **Configuration File Corruption**: Invalid JSON breaking configuration loading
- **Permission Denied**: File system access issues for configuration storage
- **Credential Loss**: Authentication tokens becoming invalid or missing
- **Server Registration Conflicts**: Duplicate server names causing conflicts
- **Schema Cache Staleness**: Outdated server schemas causing validation errors

### Code Generation Edge Cases
- **Schema Complexity**: Complex nested schemas causing generation failures
- **Naming Conflicts**: Generated code having identifier collisions
- **Template Errors**: Custom generation templates containing syntax errors
- **Output Path Issues**: Invalid or inaccessible generation destination paths
- **Type System Limitations**: Schema features not representable in TypeScript

## Technical Debt

### Protocol Implementation Completeness
- Basic MCP protocol support without advanced features
- Limited error handling for protocol edge cases
- No support for streaming or long-running tool calls
- Missing implementation of optional MCP capabilities

### Authentication and Security
- Basic OAuth implementation without robust error recovery
- Credential storage without encryption or secure keyring integration
- No support for advanced authentication methods
- Limited audit trail for authentication events

### Performance and Scalability
- No connection pooling for HTTP server communications
- Basic caching without intelligent invalidation strategies
- Sequential tool calls without concurrent execution optimization
- Memory usage scales poorly with large server configurations

### User Experience and Documentation
- Command-line interface learning curve for new users
- Limited interactive features for server discovery
- Basic error messages without troubleshooting guidance
- No graphical interface for complex configurations

## Dependencies

### Core Dependencies
- **mcporter Binary**: Primary CLI tool for MCP operations
- **Node.js Runtime**: JavaScript runtime for CLI execution
- **MCP Protocol Libraries**: Protocol implementation and communication

### Network and Communication Dependencies
- **HTTP Client**: For HTTP-based MCP server communication
- **Process Management**: For stdio server execution and lifecycle
- **WebSocket Support**: For advanced MCP transport methods
- **JSON Processing**: Message serialization and deserialization

### Authentication Dependencies
- **OAuth2 Client**: Authentication flow implementation
- **Credential Storage**: Secure token and secret management
- **Browser Integration**: OAuth consent flow handling
- **Certificate Validation**: SSL/TLS certificate verification

### Code Generation Dependencies
- **Template Engine**: Code generation template processing
- **TypeScript Compiler**: Type checking and compilation validation
- **AST Manipulation**: Code structure analysis and generation
- **File System Operations**: Generated code writing and organization

### Configuration Dependencies
- **JSON Schema Validation**: Configuration format validation
- **File Watching**: Configuration file change detection
- **Path Resolution**: Configuration file and server location handling
- **Environment Variables**: Runtime configuration override

### Optional Dependencies
- **Daemon Process**: Background service for persistent server connections
- **CLI Framework**: Enhanced command-line interface features
- **Logging Infrastructure**: Structured logging and debugging support
- **Monitoring Tools**: Server health and performance tracking