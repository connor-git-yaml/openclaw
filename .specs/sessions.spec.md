# Sessions Module Specification

## Intent
The Sessions module manages conversation lifecycle, state persistence, and context management across all agent interactions. It provides session creation, message storage, state synchronization, and cleanup services. The module ensures continuity of conversations, handles session routing between agents and channels, and maintains conversation history with proper serialization and recovery mechanisms.

## Interface

### Core Types
```typescript
export type SessionKey = {
  agentId: string;
  sessionId: string;
  channel?: string;
  accountId?: string;
}

export type SessionEntry = {
  key: SessionKey;
  createdAt: number;
  lastActivity: number;
  messageCount: number;
  status: SessionStatus;
  metadata: SessionMetadata;
  transcript: MessageEntry[];
}

export type SessionStatus = "starting" | "active" | "paused" | "stopping" | "ended";
```

### Key Functions
- `loadSessionStore(sessionKey)` - Load session state and configuration
- `updateSessionStore(sessionKey, updates)` - Persist session changes
- `resolveSessionFilePath(sessionKey)` - Get session storage path
- `loadSessionEntry(sessionKey)` - Load complete session data
- `writeSessionMessages(sessionKey, messages)` - Append to transcript
- `clearSessionQueues(sessionKey)` - Clean up pending operations
- `resetSession(sessionKey)` - Reset session state to initial

### Session Lifecycle Events
- **Creation**: Initialize new session with metadata
- **Message Append**: Add new messages to transcript
- **State Update**: Modify session status or metadata
- **Pause/Resume**: Temporarily suspend session activity
- **Cleanup**: Remove inactive sessions and free resources

## Business Logic

### Session Creation Process
1. **Key Generation**: Create unique session identifier from agent/channel/user
2. **Directory Setup**: Ensure session storage directory exists
3. **Metadata Initialization**: Set creation timestamp, status, and initial config
4. **Transcript Setup**: Create empty message history
5. **State Registration**: Add session to active session registry
6. **Event Emission**: Notify listeners of new session creation

### Message Processing Flow
1. **Message Validation**: Verify message format and required fields
2. **Transcript Append**: Add message to session conversation history
3. **Context Maintenance**: Prune old messages if context limits exceeded
4. **State Updates**: Update last activity timestamp and message count
5. **Persistence**: Write changes to storage with atomic operations
6. **Event Notification**: Emit session update events

### Session Routing Logic
- **Agent Binding**: Route sessions to appropriate agent instances
- **Channel Mapping**: Maintain session-to-channel associations
- **Account Isolation**: Separate sessions by user account
- **Subagent Hierarchy**: Parent-child session relationships
- **Cross-Context**: Enable controlled inter-session communication

### State Synchronization
- **Optimistic Updates**: Apply changes immediately with rollback capability
- **Conflict Resolution**: Handle concurrent modifications with last-write-wins
- **Periodic Sync**: Regular persistence of in-memory state
- **Recovery Mode**: Restore sessions from storage on startup
- **Integrity Checks**: Validate session data consistency

## Data Structures

### Session Store
```typescript
interface SessionStore {
  sessions: Map<string, SessionEntry>;
  activeConnections: Map<string, WebSocket[]>;
  messageQueues: Map<string, MessageQueue>;
  lastCleanup: number;
  totalSessions: number;
  activeSessionCount: number;
}
```

### Message Entry
```typescript
interface MessageEntry {
  id: string;
  timestamp: number;
  role: "user" | "assistant" | "system" | "tool";
  content: MessageContent;
  metadata?: MessageMetadata;
  toolCalls?: ToolCall[];
  attachments?: Attachment[];
}
```

### Session Metadata
```typescript
interface SessionMetadata {
  agentId: string;
  channelId: string;
  accountId: string;
  userId?: string;
  parentSession?: string;
  subagents: string[];
  modelRef: string;
  tokensUsed: number;
  lastModelCall: number;
  customFields: Record<string, any>;
}
```

### Message Queue
```typescript
interface MessageQueue {
  sessionKey: string;
  pending: QueuedMessage[];
  processing: QueuedMessage[];
  maxSize: number;
  dropPolicy: "oldest" | "newest" | "priority";
}
```

## Constraints

### Storage Limits
- **Maximum Session Size**: 100MB per session transcript
- **Message Count Limit**: 10,000 messages per session before archiving
- **Active Session Limit**: 1,000 concurrent sessions per gateway
- **Disk Usage**: Configurable storage quota with cleanup policies
- **Backup Retention**: 30-day session backup retention policy

### Performance Boundaries
- **Message Throughput**: 1,000 messages per second per session
- **Sync Frequency**: Maximum 10 state syncs per second per session
- **Memory Usage**: 50MB per active session in memory
- **Query Performance**: Sub-100ms session lookup time
- **Concurrent Operations**: 100 concurrent session operations

### Data Integrity
- **Atomic Writes**: Session updates must be atomic and consistent
- **Backup Validation**: Regular integrity checks on session backups
- **Corruption Recovery**: Ability to rebuild from message logs
- **Version Compatibility**: Support for session format migrations
- **Cross-Platform**: Consistent behavior across operating systems

## Edge Cases

### Storage Failures
- **Disk Full**: Graceful handling with emergency cleanup
- **Permission Errors**: Fallback to read-only mode
- **File Corruption**: Attempt recovery from backup copies
- **Network Storage**: Handle network filesystem disconnections
- **Concurrent Access**: File locking to prevent corruption

### Session State Conflicts
- **Concurrent Modifications**: Last-write-wins with conflict detection
- **Orphaned Sessions**: Cleanup sessions with missing agents
- **Duplicate Keys**: Handle session key collisions gracefully
- **State Inconsistency**: Detect and repair inconsistent session state
- **Memory vs Disk**: Reconcile in-memory and persisted state

### Message Processing Edge Cases
- **Oversized Messages**: Handle messages exceeding size limits
- **Malformed Content**: Graceful parsing of invalid message formats
- **Encoding Issues**: Proper UTF-8 handling and validation
- **Attachment Failures**: Handle missing or corrupt attachments
- **Tool Call Errors**: Process failed tool executions properly

### Session Lifecycle Issues
- **Unexpected Termination**: Recover sessions after crash
- **Resource Exhaustion**: Handle out-of-memory conditions
- **Network Partitions**: Maintain state during network splits
- **Time Drift**: Handle system clock changes gracefully
- **Migration Failures**: Rollback failed session migrations

## Technical Debt

### Current Issues
- **File-Based Storage**: Performance limitations with large session counts
- **Global Session Registry**: Potential memory bloat with long-running gateway
- **Synchronous I/O**: Blocking operations impact performance
- **Message Format Versioning**: Lack of proper schema evolution
- **Error Handling**: Inconsistent error recovery patterns

### Performance Bottlenecks
- **Sequential Message Writes**: Could benefit from batch operations
- **Full Session Loading**: Loading entire transcripts for metadata updates
- **No Indexing**: Slow session searches without proper indexing
- **Memory Leaks**: Potential session cleanup issues
- **Disk I/O**: Frequent sync operations impact performance

### Maintenance Issues
- **Configuration Sprawl**: Session settings scattered across modules
- **Testing Complexity**: Difficult to test edge cases reliably
- **Documentation**: Missing comprehensive session management docs
- **Monitoring**: Insufficient session health monitoring
- **Debugging**: Limited session debugging tools

### Missing Features
- **Session Analytics**: Comprehensive usage and performance metrics
- **Hot Backup**: Live backup without service interruption
- **Compression**: Transcript compression for long sessions
- **Sharding**: Distribute sessions across multiple storage nodes
- **Real-time Sync**: Live synchronization across gateway instances

## Dependencies

### Core Dependencies
- **fs/promises**: Asynchronous file system operations
- **path**: File path manipulation and resolution
- **crypto**: Session key hashing and validation
- **events**: Event emission for session lifecycle
- **worker_threads**: Background processing for heavy operations

### Internal Dependencies
- **config**: Session configuration and validation
- **agents**: Agent lifecycle and session binding
- **channels**: Channel-specific session routing
- **gateway**: WebSocket connection management
- **tools**: Tool execution context and results

### Storage Dependencies
- **SQLite**: Optional structured storage backend
- **Redis**: Optional session state caching
- **File System**: Primary storage for session transcripts
- **Backup Systems**: Integration with backup solutions
- **Compression**: Optional transcript compression

### Optional Dependencies
- **Memory Search**: Vector search integration for session content
- **Analytics**: Session usage analytics and reporting
- **Monitoring**: Session health monitoring and alerting
- **Migration Tools**: Session format migration utilities
- **Debugging Tools**: Session inspection and debugging utilities