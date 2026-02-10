# Oracle Skill Specification

## Intent

Provide comprehensive AI-powered code analysis and assistance through the Oracle CLI tool. Enables one-shot AI queries with full repository context, file bundling, and multi-model engine support for complex development tasks, debugging, and code review scenarios.

## Interface

### Core Operations
- **Context-Aware Queries**: AI assistance with complete repository context via file bundling
- **Multi-Engine Support**: API-based and browser-based AI model access
- **Session Management**: Long-running queries with persistent session storage and reattachment
- **File Processing**: Intelligent file selection, glob patterns, and exclusion handling

### Command Structure
- `oracle --dry-run -p "PROMPT" --file "src/**"` - Preview without execution
- `oracle --engine browser --model gpt-5.2-pro -p "TASK" --file "src/**"` - Browser execution
- `oracle --render --copy -p "TASK" --file "src/**"` - Manual paste fallback
- `oracle session ID --render` - Reattach to stored sessions
- `oracle status --hours 72` - List recent sessions

### Engine Selection
- **API Engine**: Direct API calls to AI models (requires API keys)
- **Browser Engine**: Browser automation for ChatGPT, Gemini interfaces
- **Auto Detection**: API when `OPENAI_API_KEY` present, browser otherwise
- **Remote Browser**: Distributed browser automation with host/client setup

### File Attachment System
- **Glob Patterns**: `--file "src/**"` with inclusion/exclusion support
- **Multiple Sources**: `--file` repeated or comma-separated entries
- **Smart Defaults**: Automatic gitignore respect and directory filtering
- **Size Limits**: 1MB file size restriction with intelligent handling

## Business Logic

### Context Bundling and Processing
1. Parse file inclusion/exclusion patterns with glob expansion
2. Respect gitignore rules and default directory exclusions
3. Bundle selected files with metadata and structure information
4. Apply size limits and content validation before submission
5. Generate comprehensive context packages for AI models

### Session Lifecycle Management
1. Create unique session identifiers with optional readable slugs
2. Store session metadata and state in `~/.oracle/sessions`
3. Handle session detachment and timeout scenarios gracefully
4. Enable session reattachment for long-running operations
5. Implement duplicate prompt detection and prevention

### Multi-Engine Coordination
1. Determine optimal engine based on available credentials and model requirements
2. Handle API-based execution with token management and rate limiting
3. Coordinate browser automation for ChatGPT and Gemini interfaces
4. Manage remote browser hosts for distributed processing
5. Handle engine-specific attachment and formatting requirements

### Intelligent File Selection
1. Apply default exclusions for common build artifacts and dependencies
2. Process glob patterns with proper expansion and filtering
3. Handle symlink detection and exclusion for security
4. Implement size-based filtering and content validation
5. Provide file selection previews and reporting capabilities

## Data Structures

### Query Context
```typescript
interface QueryContext {
  prompt: string;              // user query/task description
  files: FileSelection[];      // selected file set
  engine: 'api' | 'browser';   // execution engine
  model?: string;              // target AI model
  slug?: string;               // readable session identifier
  attachmentMode: 'auto' | 'never' | 'always';
  dryRun: boolean;             // preview mode flag
}
```

### File Selection
```typescript
interface FileSelection {
  patterns: string[];          // file glob patterns
  inclusions: string[];        // explicitly included paths
  exclusions: string[];        // explicitly excluded paths
  resolvedFiles: FileInfo[];   // final file set
  totalSize: number;           // aggregate file size
  tokenEstimate: number;       // estimated token count
}

interface FileInfo {
  path: string;                // relative file path
  size: number;                // file size in bytes
  mimeType: string;            // detected content type
  isText: boolean;             // text vs binary classification
  lastModified: Date;          // file modification timestamp
}
```

### Session State
```typescript
interface OracleSession {
  id: string;                  // unique session identifier
  slug?: string;               // user-friendly name
  created: Date;               // session creation timestamp
  status: 'running' | 'completed' | 'detached' | 'failed';
  engine: string;              // execution engine used
  model?: string;              // AI model specification
  context: QueryContext;       // original query context
  result?: SessionResult;      // completion result
  attachmentPath?: string;     // stored attachment location
  browserSession?: BrowserSessionInfo;
}
```

### Engine Configuration
```typescript
interface EngineConfig {
  type: 'api' | 'browser' | 'remote-browser';
  apiKeys: Record<string, string>; // available API credentials
  browserConfig: {
    attachments: 'auto' | 'never' | 'always';
    timeout: number;           // browser operation timeout
    headless: boolean;         // headless browser mode
  };
  remoteConfig?: {
    host: string;              // remote browser host
    port: number;              // remote browser port
    token: string;             // authentication token
  };
}
```

### Processing Result
```typescript
interface SessionResult {
  success: boolean;            // operation completion status
  response?: string;           // AI model response
  error?: string;              // error message if failed
  processingTime: number;      // total processing duration
  tokenUsage?: TokenUsage;     // API token consumption
  attachments: string[];       // generated attachments
  metadata: ResultMetadata;    // additional result information
}

interface TokenUsage {
  prompt: number;              // input tokens consumed
  completion: number;          // output tokens generated
  total: number;               // total token count
  cost?: number;               // estimated cost in USD
}
```

### File Processing Configuration
```typescript
interface FileProcessingConfig {
  maxFileSize: number;         // maximum individual file size (1MB)
  defaultExclusions: string[]; // default ignored directories
  respectGitignore: boolean;   // honor .gitignore rules
  followSymlinks: boolean;     // process symbolic links
  includeDotfiles: boolean;    // include hidden files
  binaryFileHandling: 'skip' | 'include' | 'metadata-only';
}
```

## Constraints

### File Processing Limitations
- **Size Limits**: Individual files limited to 1MB maximum
- **Binary Files**: Non-text files typically excluded from processing
- **Symlinks**: Symbolic links not followed for security reasons
- **Gitignore Respect**: Pattern expansion honors .gitignore rules

### Session and Storage Constraints
- **Local Storage**: Sessions stored in `~/.oracle/sessions` directory
- **Disk Space**: Session storage consumes local disk space
- **Session Lifetime**: No automatic session cleanup or expiration
- **Concurrent Limits**: Limited concurrent session support

### Engine-Specific Limitations
- **API Dependencies**: API engine requires valid API keys and quota
- **Browser Dependencies**: Browser engine requires GUI environment
- **Model Availability**: Specific models limited to certain engines
- **Network Connectivity**: All engines require stable internet access

### Token and Cost Management
- **Token Limits**: Large context may exceed model token limits
- **Cost Control**: No automatic cost limiting or budget controls
- **Rate Limiting**: API engines subject to provider rate limits
- **Context Windows**: Model-specific context window restrictions

## Edge Cases

### File Selection and Processing Issues
- **Large Repository**: Very large codebases exceeding practical token limits
- **Binary Content**: Binary files accidentally included causing processing issues
- **Permission Denied**: File system permissions preventing file access
- **Glob Expansion**: Complex glob patterns causing unexpected file inclusion
- **Symlink Loops**: Circular symbolic links causing infinite expansion

### Session Management Complications
- **Session Storage Full**: Disk space exhaustion preventing session storage
- **Session Corruption**: Stored session data becoming corrupted or unreadable
- **Concurrent Sessions**: Multiple oracle instances causing session conflicts
- **Home Directory Issues**: Custom `ORACLE_HOME_DIR` not accessible
- **Session ID Conflicts**: Duplicate session identifiers causing confusion

### Engine and Model Problems
- **API Key Issues**: Invalid, expired, or insufficient API credentials
- **Browser Automation Failures**: Browser automation breaking due to site changes
- **Model Unavailability**: Requested AI models temporarily offline or restricted
- **Network Timeout**: Long-running operations timing out during processing
- **Engine Switching**: Fallback between engines causing context loss

### Context and Query Complications
- **Context Overflow**: File bundle exceeding model context windows
- **Duplicate Prompts**: Identical queries causing unnecessary duplicate runs
- **Prompt Injection**: Malicious content in files affecting AI responses
- **Model Confusion**: Large context causing model response degradation
- **Token Estimation**: Inaccurate token counting causing unexpected costs

## Technical Debt

### Error Handling and Recovery
- Basic error reporting without specific guidance for resolution
- Limited retry mechanisms for transient failures
- No graceful degradation when preferred engines unavailable
- Insufficient error context for debugging complex failures

### Session Management Complexity
- Manual session cleanup without automatic expiration
- No session sharing or collaboration capabilities
- Basic session organization without categorization or search
- Limited session metadata for tracking and analysis

### File Processing Optimization
- Sequential file processing without parallel optimization
- Basic content type detection without advanced analysis
- No intelligent file prioritization based on relevance
- Limited preprocessing options for code analysis

### User Experience Limitations
- Command-line only interface without graphical components
- Complex configuration options requiring deep understanding
- No interactive file selection or query refinement
- Basic progress reporting for long-running operations

## Dependencies

### Core Dependencies
- **Oracle CLI Binary**: Primary tool for AI query orchestration
- **Node.js Runtime**: JavaScript execution environment
- **SQLite**: Database backend for session storage
- **File System Access**: Read/write permissions for files and sessions

### AI Engine Dependencies
- **API Providers**: OpenAI, Anthropic, Google, and other AI service APIs
- **Browser Automation**: Puppeteer or similar for browser engine
- **HTTP Clients**: Network communication with AI services
- **Authentication**: API key management and secure storage

### File Processing Dependencies
- **Glob Parsing**: Pattern matching for file selection
- **Gitignore Processing**: .gitignore rule interpretation and application
- **MIME Type Detection**: Content type identification for files
- **Path Manipulation**: Cross-platform file path handling

### System Dependencies
- **Network Connectivity**: Internet access for AI model communication
- **Browser Environment**: GUI environment for browser automation (when needed)
- **Process Management**: Session lifecycle and background task management
- **Configuration Storage**: Persistent configuration and credential storage

### Optional Dependencies
- **Remote Browser**: Distributed browser automation infrastructure
- **Cost Tracking**: Token usage and cost monitoring tools
- **Integration**: IDE plugins and development workflow integration
- **Monitoring**: Session and performance monitoring capabilities

### Development Dependencies
- **Package Management**: npm for installation and dependency management
- **Build Tools**: Compilation and distribution tools
- **Testing Framework**: Validation of AI query accuracy and functionality
- **Documentation**: Usage guides and best practices documentation