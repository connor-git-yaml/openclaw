# Extension Specification: BlueBubbles

## Intent

BlueBubbles extension enables iMessage communication through the BlueBubbles macOS app via REST API. It provides full messaging capabilities including sending/receiving messages, group chat management, media attachments, reactions, and contact management. The extension acts as a channel plugin bridging OpenClaw's messaging infrastructure with Apple's iMessage ecosystem through the BlueBubbles intermediary.

## Interface

### Plugin Registration
- **ID**: `bluebubbles`
- **Name**: BlueBubbles
- **Type**: Channel Plugin
- **Entry Point**: `index.ts` exports plugin definition with register() function
- **API Surface**: Registers channel plugin and HTTP webhook handler

### Channel Plugin Methods
- `sendMessage()` - Send text/media messages to individuals or groups
- `editMessage()` - Edit previously sent messages (macOS 16+ required)
- `unsendMessage()` - Delete/retract sent messages
- `sendReaction()` - Add/remove emoji reactions to messages
- `manageGroup()` - Create, rename groups, manage participants
- `uploadAttachment()` - Send media files (images, videos, documents)

### HTTP Endpoints
- **Webhook**: Receives incoming messages from BlueBubbles server
- **Path**: Configurable per account (e.g., `/bluebubbles-webhook`)
- **Method**: POST with JSON message payloads

## Business Logic

### Authentication & Connection
- Requires BlueBubbles server URL and password configuration
- Maintains persistent connection for real-time message receiving
- Supports multiple account configurations with independent settings
- Automatic reconnection handling on connection failures

### Message Processing
- **Outbound**: Converts OpenClaw message format to BlueBubbles API calls
- **Inbound**: Processes webhooks, normalizes to OpenClaw message events
- **Target Resolution**: Maps phone numbers/emails to chat GUIDs
- **Group Handling**: Manages group metadata, participant lists, permissions

### Policy Enforcement
- **DM Policy**: `pairing`, `allowlist`, `open`, or `disabled`
- **Group Policy**: Controls group participation and mention requirements
- **Tool Policy**: Per-group tool access controls
- **Capability Tags**: Runtime guidance for agent behavior

### State Management
- Tracks active chats and participants
- Maintains message ID mapping for edits/deletes
- Caches contact information and group metadata
- Handles pairing approval workflow for new contacts

## Data Structures

### Configuration Schema
```typescript
BlueBubblesAccountConfig {
  name?: string;
  capabilities?: string[];
  enabled?: boolean;
  serverUrl?: string;
  password?: string;
  webhookPath?: string;
  dmPolicy?: DmPolicy;
  allowFrom?: Array<string | number>;
  groupConfig?: Record<string, BlueBubblesGroupConfig>;
}
```

### Message Types
```typescript
BlueBubblesSendTarget {
  accountId: string;
  targetId: string; // phone number, email, or chat GUID
}

BlueBubblesMessage {
  guid: string;
  text?: string;
  attachments?: BlueBubblesAttachment[];
  isFromMe: boolean;
  chatGuid: string;
  dateCreated: number;
}
```

### Runtime State
- Account connection status and health metrics
- Active chat sessions and participant mappings
- Pending message operations and retry queues
- Webhook processing statistics

## Constraints

### Technical Limitations
- **macOS Only**: Requires BlueBubbles macOS app for iMessage access
- **Message Editing**: Only available on macOS 16+ (requires modern iMessage features)
- **Media Size**: Limited by iMessage attachment size restrictions
- **Rate Limiting**: Subject to iMessage service rate limits
- **Network Dependency**: Requires stable connection to BlueBubbles server

### Operational Constraints
- **Apple ID**: Requires signed-in Apple ID on host macOS device
- **Server Requirements**: BlueBubbles server must be running and accessible
- **Firewall**: Webhook endpoint must be reachable from BlueBubbles server
- **Permissions**: May require macOS accessibility permissions

### Policy Restrictions
- DM policy enforcement may block legitimate contacts
- Group mention requirements could limit responsiveness
- Tool policy restrictions apply per-group configuration

## Edge Cases

### Connection Failures
- **Server Unreachable**: Graceful degradation with retry logic
- **Authentication Errors**: Clear error reporting with re-auth prompts
- **Network Interruption**: Message queuing and retry mechanisms
- **Webhook Failures**: Dead letter queue for failed webhook deliveries

### Message Handling
- **Duplicate Messages**: Deduplication based on message GUIDs
- **Out-of-Order Delivery**: Timestamp-based ordering and gap detection
- **Large Attachments**: Chunked upload handling with progress tracking
- **Unsupported Media**: Fallback to generic attachment handling

### Group Management
- **Participant Changes**: Real-time sync with group membership updates
- **Permission Conflicts**: Graceful handling of insufficient permissions
- **Group Naming**: Unicode and special character handling in group names

### Target Resolution
- **Ambiguous Contacts**: Multiple matches resolved via recency/frequency
- **Unknown Numbers**: Dynamic contact discovery and pairing workflow
- **International Formats**: Phone number normalization and validation

## Technical Debt

### Code Organization
- Scattered configuration validation across multiple files
- Inconsistent error handling patterns between modules
- Duplicate target parsing logic in send.ts and targets.ts

### Performance Issues
- Synchronous webhook processing could block under high load
- Missing connection pooling for multiple BlueBubbles accounts
- Inefficient message history lookups for large conversations

### Testing Coverage
- Limited integration tests for multi-account scenarios
- Missing error simulation tests for network failures
- Insufficient edge case coverage for group management operations

### API Design
- Inconsistent naming conventions between internal and external APIs
- Missing standardized retry policies across operations
- Lack of structured logging for debugging complex scenarios

## Dependencies

### External Dependencies
- **BlueBubbles macOS App**: Core messaging infrastructure
- **BlueBubbles Server**: REST API endpoint for messaging operations
- **iMessage Service**: Apple's messaging platform (via BlueBubbles)

### Internal Dependencies
- **openclaw/plugin-sdk**: Core plugin framework and utilities
- **Channel Infrastructure**: Message routing and account management
- **HTTP Server**: Webhook processing and request handling
- **Configuration System**: Account settings and policy enforcement

### Runtime Requirements
- **macOS Host**: Required for BlueBubbles app execution
- **Network Connectivity**: Stable connection to BlueBubbles server
- **Storage**: Message history and configuration persistence
- **Permissions**: macOS accessibility and network permissions