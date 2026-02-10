# Extension Specification: Lobster

## Intent

Lobster extension provides integration with the Lobster automation tool, enabling OpenClaw to execute Lobster scripts for complex automation tasks. Lobster is a specialized automation framework for web interactions, screen scraping, and browser automation. The extension acts as a secure wrapper around the Lobster executable, providing approval workflows, output processing, and sandboxing controls for safe automation execution.

## Interface

### Plugin Registration
- **Type**: Tool Extension
- **Sandboxing**: Disabled in sandboxed environments for security
- **Optional Registration**: Marked as optional tool (non-critical)
- **Conditional Loading**: Only loads when not in sandboxed context

### Tool Definition
- **Tool Name**: `lobster`
- **Execution Mode**: External process spawning
- **Approval Workflow**: Interactive approval for sensitive operations
- **Output Processing**: Structured JSON output parsing

### Input Parameters
```typescript
LobsterParams {
  script: string; // Lobster script content
  args?: string[]; // Command-line arguments
  workingDir?: string; // Execution directory
  timeout?: number; // Maximum execution time
  lobsterPath?: string; // Custom Lobster executable path
}
```

### Security Controls
- **Executable Validation**: Strict validation of Lobster executable path
- **Sandboxing Detection**: Automatic disabling in sandboxed environments
- **Approval Requirements**: Interactive approval for potentially sensitive operations
- **Path Restrictions**: Prevents execution of arbitrary binaries

## Business Logic

### Executable Resolution
- **Path Detection**: Automatic detection of Lobster installation
- **Security Validation**: Ensures only Lobster executable can be run
- **Absolute Path Requirements**: Custom paths must be absolute for security
- **Installation Verification**: Validates Lobster is properly installed

### Process Management
- **Child Process Spawning**: Manages Lobster subprocess execution
- **Timeout Handling**: Enforces execution time limits
- **Resource Management**: Monitors and controls resource usage
- **Output Streaming**: Real-time output capture and processing

### Approval Workflow
- **Interactive Approval**: Prompts user for approval of sensitive operations
- **Resume Token System**: Supports paused execution with resume capability
- **Approval Context**: Provides detailed information about requested operations
- **Cancellation Support**: Allows users to cancel pending operations

### Output Processing
- **JSON Parsing**: Processes structured Lobster output
- **Error Handling**: Distinguishes between success and error responses
- **Status Interpretation**: Handles various execution status codes
- **Data Extraction**: Extracts meaningful results from Lobster output

## Data Structures

### Lobster Envelope
```typescript
LobsterEnvelope = {
  ok: true;
  status: "ok" | "needs_approval" | "cancelled";
  output: unknown[];
  requiresApproval?: {
    type: "approval_request";
    prompt: string;
    items: unknown[];
    resumeToken?: string;
  };
} | {
  ok: false;
  error: {
    type?: string;
    message: string;
  };
}
```

### Execution Context
```typescript
ExecutionContext {
  workingDir: string; // Script execution directory
  executable: string; // Resolved Lobster executable path
  args: string[]; // Complete argument list
  timeout: number; // Execution timeout in milliseconds
}
```

### Approval Request
```typescript
ApprovalRequest {
  prompt: string; // User-facing approval prompt
  items: unknown[]; // Details of requested operations
  resumeToken?: string; // Token for resuming execution
  context: ApprovalContext; // Additional context information
}
```

## Constraints

### Security Restrictions
- **Sandboxing Prohibition**: Cannot run in sandboxed environments
- **Executable Validation**: Strict validation of executable paths
- **Path Security**: Only absolute paths allowed for custom executables
- **Binary Name Enforcement**: Only 'lobster' executable name permitted

### Technical Limitations
- **External Dependency**: Requires Lobster to be installed on system
- **Platform Compatibility**: Limited to platforms supporting Lobster
- **Process Overhead**: Additional overhead from subprocess management
- **Timeout Constraints**: Long-running scripts may exceed timeout limits

### Environment Requirements
- **Installation Dependency**: Lobster must be properly installed and accessible
- **Permissions**: Requires appropriate file system and execution permissions
- **Working Directory**: Must have read/write access to working directory
- **Network Access**: May require network access for web automation scripts

### Resource Constraints
- **Memory Usage**: Limited by system memory and Lobster's requirements
- **CPU Resources**: Intensive automation may consume significant CPU
- **Timeout Limits**: Scripts must complete within configured time limits
- **Concurrent Execution**: No built-in support for parallel Lobster instances

## Edge Cases

### Installation Issues
- **Missing Lobster**: Lobster executable not found in PATH or specified location
- **Version Incompatibility**: Incompatible Lobster version causing failures
- **Permission Denied**: Insufficient permissions to execute Lobster
- **Corrupted Installation**: Damaged or incomplete Lobster installation

### Execution Failures
- **Script Errors**: Malformed or erroneous Lobster scripts
- **Timeout Exceeded**: Long-running scripts exceeding configured limits
- **Resource Exhaustion**: Scripts consuming excessive system resources
- **External Dependencies**: Missing dependencies for web automation

### Approval Workflow Issues
- **User Unavailability**: Required approvals when user is not available
- **Approval Timeout**: User taking too long to respond to approval requests
- **Resume Token Corruption**: Invalid or expired resume tokens
- **Approval Rejection**: User rejecting required operations

### Output Processing Problems
- **Invalid JSON**: Malformed JSON output from Lobster
- **Unexpected Format**: Output format changes in different Lobster versions
- **Large Output**: Extremely large output exceeding memory limits
- **Encoding Issues**: Character encoding problems in output

## Technical Debt

### Security Considerations
- **Path Validation**: Complex path validation logic could be bypassed
- **Sandboxing Detection**: Sandboxing detection may not cover all scenarios
- **Approval Bypass**: Potential for approval workflow bypassing
- **Executable Security**: Risk of path traversal or symbolic link attacks

### Code Organization
- **Monolithic Implementation**: Large tool implementation combining multiple concerns
- **Error Handling**: Inconsistent error handling across different failure modes
- **Configuration Management**: Limited configuration options and validation
- **Testing Challenges**: Difficult to unit test due to external executable dependency

### Process Management
- **Resource Cleanup**: Potential for zombie processes or resource leaks
- **Signal Handling**: Limited signal handling for process termination
- **Output Buffering**: Potential memory issues with large output streams
- **Timeout Implementation**: Simple timeout may not handle all edge cases

### Approval System
- **Approval State Management**: Complex state management for pending approvals
- **Resume Logic**: Resume token system could be more robust
- **User Experience**: Approval prompts may lack sufficient context
- **Audit Trail**: Limited logging for approval decisions and outcomes

## Dependencies

### External Dependencies
- **Lobster Executable**: External Lobster automation tool installation
- **Node.js Child Process**: Built-in Node.js process spawning capabilities
- **File System Access**: Node.js fs module for path validation
- **TypeBox**: Runtime type validation and schema generation

### Internal Dependencies
- **OpenClaw Plugin SDK**: Core plugin framework and tool registration
- **Approval System**: User approval workflow infrastructure
- **Process Management**: Subprocess execution and monitoring
- **Security Framework**: Sandboxing detection and security controls

### Runtime Requirements
- **Lobster Installation**: Functional Lobster automation tool installation
- **System Permissions**: Execution permissions and file system access
- **Memory Resources**: Sufficient memory for subprocess execution and output processing
- **Network Connectivity**: May require network access for web automation tasks