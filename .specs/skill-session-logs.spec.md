# Session Logs Skill Specification

## Intent

Provide comprehensive search and analysis capabilities for OpenClaw conversation history stored in JSONL session files. Enables historical context retrieval, conversation analytics, cost tracking, and tool usage analysis for understanding past interactions and decision-making patterns.

## Interface

### Core Operations
- **Historical Search**: Find conversations containing specific keywords or phrases
- **Session Analytics**: Analyze conversation patterns, costs, and tool usage
- **Temporal Queries**: Search conversations by date ranges and timeframes
- **Cost Analysis**: Track API usage costs across sessions and time periods
- **Content Extraction**: Extract user messages, assistant responses, and tool interactions

### Data Location
- **Base Directory**: `~/.openclaw/agents/<agentId>/sessions/`
- **Session Index**: `sessions.json` - Maps session keys to session IDs
- **Conversation Files**: `<session-id>.jsonl` - Complete conversation transcripts

### Query Patterns
- **Text Search**: `rg "phrase" sessions/*.jsonl` - Cross-session phrase search
- **JSON Filtering**: `jq 'select(.message.role == "user")' file.jsonl` - Role-based filtering
- **Date Filtering**: Extract sessions from specific dates or ranges
- **Cost Aggregation**: Sum usage costs across sessions or time periods

### Common Analysis Tasks
- List sessions by date and size for overview
- Extract user questions and assistant responses
- Analyze tool usage patterns and frequency
- Calculate daily/monthly cost summaries
- Search historical context for current conversations

## Business Logic

### Session File Structure and Parsing
1. Parse JSONL files with one JSON object per message line
2. Handle session metadata and conversation message structure
3. Extract message content filtering by type and role
4. Process timestamp information for temporal analysis
5. Handle file corruption and partial session recovery

### Search and Filtering Engine
1. Implement text-based search across multiple session files
2. Apply JSON-based filtering for structured message queries
3. Support regular expression search with case sensitivity options
4. Combine temporal and content-based search criteria
5. Handle large session files with streaming and pagination

### Analytics and Aggregation
1. Calculate conversation statistics (message counts, duration, participants)
2. Aggregate cost data across sessions and time periods
3. Analyze tool usage patterns and effectiveness
4. Generate conversation trend analysis and insights
5. Export analytics data for external reporting

### Historical Context Integration
1. Link current conversations to relevant historical context
2. Identify conversation threads and related discussions
3. Extract decision points and outcomes from past sessions
4. Support context reconstruction for follow-up conversations
5. Maintain conversation continuity across session boundaries

## Data Structures

### Session Message
```typescript
interface SessionMessage {
  type: 'session' | 'message';    // record type identifier
  timestamp: string;              // ISO timestamp
  message: MessageContent;        // message data
  usage?: UsageMetrics;           // cost and token usage
  sessionId: string;              // session identifier
  agentId: string;                // agent identifier
}
```

### Message Content
```typescript
interface MessageContent {
  role: 'user' | 'assistant' | 'toolResult'; // message role
  content: ContentBlock[];       // message content blocks
  messageId?: string;            // unique message identifier
  parentMessageId?: string;      // reply chain reference
  provider?: string;             // chat platform (discord, whatsapp, etc.)
}

interface ContentBlock {
  type: 'text' | 'toolCall' | 'toolResult' | 'thinking';
  text?: string;                 // text content
  name?: string;                 // tool name (for toolCall)
  input?: any;                   // tool input parameters
  output?: any;                  // tool output result
}
```

### Usage Metrics
```typescript
interface UsageMetrics {
  cost: {
    total: number;               // total cost in USD
    input?: number;              // input token cost
    output?: number;             // output token cost
    reasoning?: number;          // reasoning token cost
  };
  tokens: {
    input: number;               // input token count
    output: number;              // output token count
    total: number;               // total token count
    reasoning?: number;          // reasoning token count
  };
  model: string;                 // AI model used
  provider: string;              // API provider
}
```

### Session Index Entry
```typescript
interface SessionIndexEntry {
  sessionKey: string;            // provider-specific session key
  sessionId: string;             // OpenClaw session identifier
  created: string;               // session creation timestamp
  lastMessage: string;           // last message timestamp
  provider: string;              // chat platform
  messageCount: number;          // total messages in session
  isActive: boolean;             // session status
}
```

### Search Query
```typescript
interface SearchQuery {
  keywords: string[];            // text search terms
  dateRange?: {
    start: Date;
    end: Date;
  };
  roles?: ('user' | 'assistant' | 'toolResult')[]; // message role filter
  contentTypes?: ('text' | 'toolCall' | 'thinking')[]; // content type filter
  sessions?: string[];           // specific session IDs
  caseSensitive: boolean;        // search case sensitivity
  useRegex: boolean;             // regular expression search
}
```

### Analytics Result
```typescript
interface SessionAnalytics {
  period: {
    start: Date;
    end: Date;
  };
  sessionCount: number;          // total sessions
  messageCount: number;          // total messages
  totalCost: number;             // aggregate cost
  averageCostPerMessage: number; // cost efficiency
  toolUsage: ToolUsageStats[];   // tool usage breakdown
  conversationPatterns: ConversationPattern[];
  dailySummary: DailySummary[];  // daily aggregation
}

interface ToolUsageStats {
  toolName: string;              // tool identifier
  useCount: number;              // usage frequency
  totalCost: number;             // tool-specific cost
  successRate: number;           // success percentage
  averageExecutionTime: number;  // performance metric
}
```

## Constraints

### File System and Storage Limitations
- **Large Files**: Session files can grow to several MB requiring efficient streaming
- **File Permissions**: Read access required to session directory and files
- **Disk Space**: Historical sessions consume significant storage over time
- **File Locking**: Concurrent access during active sessions may cause conflicts

### JSON Processing Constraints
- **Line-by-Line Processing**: JSONL format requires streaming JSON parser
- **Memory Usage**: Large session analysis may consume significant RAM
- **JSON Validity**: Malformed JSON lines from interrupted sessions
- **Character Encoding**: Unicode and special character handling in message content

### Search Performance Limitations
- **Text Search Speed**: Large history searches may be slow without indexing
- **Regular Expression Complexity**: Complex regex patterns affecting performance
- **Cross-Session Search**: Searching multiple large files simultaneously resource-intensive
- **Real-time Constraints**: Search during active conversations may conflict

### Privacy and Security Considerations
- **Sensitive Content**: Session logs may contain personal or confidential information
- **Data Retention**: Long-term storage of conversation history privacy implications
- **Access Control**: No built-in access restrictions for session log files
- **Data Export**: Historical data extraction may expose sensitive information

## Edge Cases

### File System and Access Issues
- **Missing Session Files**: Referenced sessions deleted or moved
- **Permission Denied**: Insufficient file system permissions for session directory
- **Corrupted JSONL**: Malformed JSON lines from interrupted writes
- **Disk Full**: Storage exhaustion preventing session log access
- **File Locking**: Active sessions preventing read access to current logs

### Search and Query Complications
- **Large Result Sets**: Searches returning overwhelming amounts of data
- **Complex JSON Structures**: Nested content making extraction difficult
- **Unicode Issues**: Special characters causing search and display problems
- **Regex Errors**: Invalid regular expressions causing query failures
- **Memory Exhaustion**: Large session analysis consuming all available RAM

### Data Integrity and Consistency
- **Partial Sessions**: Incomplete session data from crashes or interruptions
- **Timestamp Issues**: Incorrect or missing timestamps affecting temporal queries
- **Session ID Conflicts**: Duplicate or invalid session identifiers
- **Message Order**: Out-of-order messages affecting conversation flow analysis
- **Missing Metadata**: Incomplete usage or cost information

### Tool and Command Failures
- **jq Errors**: JSON query syntax errors or processing failures
- **ripgrep Failures**: Text search tool errors or pattern matching issues
- **Shell Command Issues**: Complex shell pipelines failing with poor error messages
- **Path Resolution**: Incorrect agent ID or session path causing file not found errors
- **Command Timeout**: Long-running analysis commands exceeding reasonable time limits

## Technical Debt

### Search and Indexing Limitations
- No pre-built search indices requiring real-time file scanning
- Basic text search without semantic or contextual understanding
- Linear search performance degrading with large conversation histories
- No search result ranking or relevance scoring

### Analytics and Reporting Gaps
- Basic statistical analysis without advanced conversation insights
- No trend analysis or pattern recognition across conversation history
- Limited visualization capabilities for conversation analytics
- Manual export and integration required for external analysis tools

### Query Interface Complexity
- Command-line only interface requiring jq and shell scripting expertise
- No user-friendly query builder or graphical interface
- Complex multi-file operations requiring advanced shell knowledge
- Limited error handling and user guidance for failed queries

### Data Management and Organization
- No automatic session archiving or retention policies
- Basic file organization without categorization or tagging
- Limited compression or optimization for long-term storage
- No data migration tools for schema changes or format updates

## Dependencies

### Core Dependencies
- **jq**: JSON command-line processor for filtering and analysis
- **ripgrep (rg)**: Fast text search tool for cross-session search
- **Shell Environment**: Bash/zsh for script execution and pipeline operations
- **File System Access**: Read permissions for session directories

### JSON Processing Dependencies
- **JSON Parser**: jq for structured JSON querying and manipulation
- **Stream Processing**: Tools for handling large JSONL files efficiently
- **Unicode Support**: Proper character encoding handling for international content
- **Memory Management**: Efficient processing of large JSON datasets

### Text Processing Dependencies
- **Regular Expression Engine**: Advanced pattern matching capabilities
- **Text Search**: ripgrep for high-performance full-text search
- **String Processing**: awk, sed, and other text manipulation tools
- **Character Encoding**: UTF-8 and unicode support for message content

### System Dependencies
- **Bash/Shell**: Command execution environment for complex queries
- **Process Management**: Background job control for long-running analysis
- **File Operations**: ls, head, tail, and other file system utilities
- **Date Processing**: Date parsing and manipulation utilities

### Optional Dependencies
- **Data Visualization**: Tools for graphing conversation trends and analytics
- **Export Tools**: CSV, JSON, or other format conversion utilities
- **Compression**: gzip, zstd for session log compression and archiving
- **Database**: SQLite or similar for indexed search and analytics

### Development Dependencies
- **Testing Framework**: Validation of search and analysis functionality
- **Performance Profiling**: Analysis of search performance and optimization
- **Documentation**: Usage examples and query pattern libraries
- **Schema Validation**: Verification of session log format consistency