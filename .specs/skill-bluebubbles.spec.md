# BlueBubbles Skill Specification

## Intent

Provide iMessage integration for OpenClaw through the BlueBubbles server, enabling full-featured messaging capabilities including send, react, edit, unsend, reply, and attachment handling. Serves as the recommended iMessage channel for personal and automated messaging workflows.

## Interface

### Core Messaging Operations
- **Send Messages**: Text and rich content to individuals and groups
- **Message Reactions**: Add/remove tapback reactions to specific messages
- **Message Management**: Edit, unsend, and reply to existing messages
- **Attachments**: Send files, photos, and media with optional captions
- **Effects**: Send messages with iMessage visual effects

### Channel Integration
All operations use the generic `message` tool with `channel: "bluebubbles"`

### Target Formats
- **Phone Numbers**: E.164 format (`+15551234567`)
- **Email Addresses**: Apple ID email addresses (`user@example.com`)
- **Chat GUIDs**: Preferred format for groups (`chat_guid:...`)

### Message Identifiers
- **Full Message IDs**: Durable across server restarts
- **Short Message IDs**: Temporary session-based references

## Business Logic

### Message Routing and Delivery
1. Accept target specification in multiple formats
2. Resolve targets to BlueBubbles chat identifiers
3. Queue messages through BlueBubbles server
4. Handle delivery confirmations and failure notifications
5. Maintain message ID mapping for subsequent operations

### Reaction System
1. Support standard iMessage tapback reactions (❤️, 👍, 👎, 😂, ‼️, ❓)
2. Apply reactions to specific messages via message ID reference
3. Handle reaction removal with toggle functionality
4. Manage reaction conflicts and updates

### Message Editing and Management
1. Edit previously sent messages with content replacement
2. Unsend messages with deletion from conversation
3. Create threaded replies linked to specific message contexts
4. Handle edit/unsend limitations based on macOS version compatibility

### Attachment Processing
1. Accept local file paths or base64-encoded content
2. Support multiple media types (photos, videos, documents)
3. Add optional captions to attachment messages
4. Handle attachment size limits and format restrictions

## Data Structures

### Message Entity
```typescript
interface BlueBubblesMessage {
  id: string;                  // full message identifier
  shortId?: string;            // session-based short ID
  chatGuid: string;            // target chat identifier
  content: string;             // message text content
  sender: string;              // sender identifier
  timestamp: Date;             // send timestamp
  isFromMe: boolean;           // sent by current user
  hasAttachments: boolean;     // contains media files
  reactionType?: string;       // tapback reaction if applicable
}
```

### Chat Context
```typescript
interface BlueBubblesChat {
  chatGuid: string;            // unique chat identifier
  displayName: string;         // chat/contact display name
  participants: string[];      // participant handles
  isGroup: boolean;            // group vs individual chat
  lastMessage?: BlueBubblesMessage;
  unreadCount: number;
}
```

### Attachment Data
```typescript
interface MessageAttachment {
  filename: string;            // original filename
  mimeType: string;            // content MIME type
  size: number;                // file size in bytes
  localPath?: string;          // local file path
  base64Data?: string;         // encoded file content
  caption?: string;            // optional caption text
}
```

### Effect Configuration
```typescript
interface MessageEffect {
  name: string;                // effect identifier
  displayName: string;         // user-friendly name
  isAvailable: boolean;        // compatibility status
  requiresSpecificContent?: boolean; // content-dependent effects
}

// Common effects: balloons, confetti, fireworks, heart, lasers, shooting-star, slam, spotlight
```

## Constraints

### Server Dependencies
- **BlueBubbles Server**: Requires configured and running BlueBubbles server
- **macOS Host**: BlueBubbles server must run on macOS with Messages.app access
- **Network Connectivity**: Persistent connection between OpenClaw and BlueBubbles server
- **Authentication**: Valid server password and webhook configuration

### Platform Limitations
- **macOS Version**: Some features (edit/unsend) depend on macOS version
- **iMessage Account**: Requires active iMessage account on server Mac
- **Apple ID**: Must be signed into iMessage service
- **Network Requirements**: Internet connectivity for message delivery

### Message Constraints
- **Size Limits**: Attachment size restrictions from iMessage service
- **Format Support**: Limited to iMessage-compatible file types
- **Effect Availability**: Visual effects depend on recipient device capabilities
- **Group Limits**: iMessage group size and permission restrictions

## Edge Cases

### Connection and Availability
- **Server Offline**: BlueBubbles server unavailable or crashed
- **Network Interruption**: Connection loss during message operations
- **Messages.app Issues**: Target Messages.app unresponsive or signed out
- **Webhook Failures**: Webhook endpoint unreachable or misconfigured

### Target Resolution Problems
- **Invalid Handles**: Phone numbers or email addresses not reachable
- **Chat GUID Conflicts**: Duplicate or changed chat identifiers
- **Contact Sync Issues**: Contacts not synchronized with server Mac
- **Group Permission Changes**: User removed from group conversation

### Message Operation Failures
- **Edit Not Supported**: Attempting edit on incompatible macOS version
- **Message Too Old**: Edit/unsend operations beyond time limits
- **Reaction Conflicts**: Multiple rapid reaction changes
- **Attachment Upload Failures**: Large files or unsupported formats

### State Synchronization Issues
- **Message ID Mismatch**: Short vs full ID conflicts during operations
- **Delivery Status Uncertainty**: Messages in pending/failed state
- **Conversation State Drift**: Server and client conversation state misalignment
- **Duplicate Message Handling**: Same message appearing multiple times

## Technical Debt

### Configuration Complexity
- Manual BlueBubbles server setup required
- Configuration drift between OpenClaw and server settings
- No automated health checking of server connection
- Limited error context for configuration issues

### Message ID Management
- Dual ID system (short/full) creates confusion
- No consistent ID resolution strategy
- Message reference validation gaps
- Limited message history persistence

### Error Handling Deficiencies
- Generic error messages for diverse failure modes
- No automatic retry logic for transient failures
- Insufficient context for debugging connection issues
- Poor user feedback for permission-related failures

### Feature Coverage Gaps
- No message search or history retrieval
- Limited conversation management (mute, archive)
- No typing indicator support
- Missing read receipt functionality

## Dependencies

### Core Dependencies
- **OpenClaw Message System**: Generic message tool and channel framework
- **BlueBubbles Server**: Third-party iMessage bridge server
- **Messages.app**: macOS native messaging application

### Configuration Dependencies
- **Gateway Configuration**: `channels.bluebubbles` section with server details
- **Server Authentication**: Password and webhook path configuration
- **Network Access**: HTTP/HTTPS connectivity to BlueBubbles server

### System Dependencies
- **macOS**: Required platform for BlueBubbles server
- **iMessage Service**: Active Apple ID and iMessage account
- **Network Infrastructure**: Stable internet connection for message delivery

### Development Dependencies
- **BlueBubbles Extension**: OpenClaw extension in `extensions/bluebubbles/`
- **Webhook Handler**: HTTP endpoint for receiving BlueBubbles events
- **JSON Processing**: Message serialization and parsing

### Optional Dependencies
- **Contact Sync**: For contact name resolution
- **Media Processing**: For attachment format conversion
- **Notification System**: For message delivery alerts
- **Logging Framework**: For debugging and monitoring