# iMsg Skill Specification

## Intent

Provide native macOS iMessage/SMS integration through direct Messages.app CLI interface. Enables message reading, sending, chat monitoring, and attachment handling for personal messaging workflows with native Apple ecosystem integration.

## Interface

### Core Operations
- **Chat Management**: List conversations and retrieve chat identifiers
- **Message History**: Access conversation history with attachments and metadata
- **Real-time Monitoring**: Watch chats for incoming messages with live updates
- **Message Sending**: Send text messages and media attachments via iMessage/SMS

### Command Structure
- `imsg chats --limit N --json` - List conversations with metadata
- `imsg history --chat-id ID --limit N --attachments --json` - Get conversation history
- `imsg watch --chat-id ID --attachments` - Monitor chat for new messages
- `imsg send --to HANDLE --text TEXT [--file PATH]` - Send messages with attachments

### Service Selection
- `--service imessage` - Force iMessage delivery
- `--service sms` - Force SMS delivery  
- `--service auto` - Automatic service selection based on recipient

### Output Formats
- JSON output (`--json`) for programmatic processing
- Human-readable text for interactive use
- Attachment inclusion (`--attachments`) for media-rich conversations

## Business Logic

### Messages.app Integration
1. Interface directly with macOS Messages.app database and APIs
2. Handle authentication and permission requests for Messages access
3. Support both iMessage and SMS protocols through unified interface
4. Manage service selection and delivery method optimization

### Chat Discovery and Management
1. Enumerate available conversations with participants and metadata
2. Map chat identifiers to human-readable conversation names
3. Handle group conversations and individual chat threads
4. Maintain conversation state and message threading

### Real-time Message Processing
1. Monitor Messages.app database for new message arrivals
2. Process incoming messages with sender identification and content extraction
3. Handle multimedia attachments and rich message content
4. Provide live streaming interface for conversation monitoring

### Message Composition and Delivery
1. Compose messages with text content and file attachments
2. Validate recipient handles and service availability
3. Handle delivery confirmation and message status tracking
4. Support multimedia message composition and attachment processing

## Data Structures

### Chat Conversation
```typescript
interface ChatConversation {
  chatId: number;              // unique chat identifier
  participants: ContactInfo[]; // conversation participants
  displayName: string;         // chat display name
  isGroup: boolean;            // group vs individual chat
  messageCount: number;        // total message count
  lastMessageDate: Date;       // most recent message timestamp
  isArchived: boolean;         // conversation archive status
  serviceType: 'iMessage' | 'SMS';
}
```

### Message Entry
```typescript
interface MessageEntry {
  id: string;                  // unique message identifier
  chatId: number;              // containing conversation
  sender: ContactInfo;         // message sender information
  content: string;             // message text content
  timestamp: Date;             // send/receive timestamp
  isFromMe: boolean;           // sent by current user
  messageType: 'text' | 'attachment' | 'reaction';
  deliveryStatus: 'delivered' | 'read' | 'pending' | 'failed';
  attachments: MessageAttachment[];
  reactions: MessageReaction[];
}
```

### Contact Information
```typescript
interface ContactInfo {
  handle: string;              // phone number or email
  displayName?: string;        // contact display name
  handleType: 'phone' | 'email';
  isContact: boolean;          // exists in Contacts app
  avatarPath?: string;         // contact photo path
}
```

### Message Attachment
```typescript
interface MessageAttachment {
  id: string;                  // attachment identifier
  filename: string;            // original filename
  mimeType: string;            // content type
  size: number;                // file size in bytes
  localPath?: string;          // local file path
  thumbnailPath?: string;      // preview image path
  isDownloaded: boolean;       // local availability status
}
```

### Send Request
```typescript
interface SendRequest {
  recipients: string[];        // target handles (phone/email)
  text: string;                // message content
  attachmentPath?: string;     // optional file attachment
  service: 'imessage' | 'sms' | 'auto';
  priority?: 'normal' | 'urgent';
}
```

## Constraints

### macOS Platform Requirements
- **Messages.app**: Must be installed, configured, and signed in
- **Full Disk Access**: Terminal/app requires Full Disk Access permission
- **Automation Permission**: Must allow automation control of Messages.app
- **Apple ID**: Valid Apple ID signed into iMessage service

### Service and Network Dependencies
- **iMessage Service**: Active iMessage account for Apple-to-Apple messaging
- **SMS Service**: Cellular connection or iPhone relay for SMS delivery
- **Internet Connectivity**: iMessage requires internet access for delivery
- **Handoff**: May require iPhone proximity for SMS relay functionality

### Permission and Security Constraints
- **System Permissions**: Multiple macOS security permissions required
- **Messages Database**: Direct access to Messages.app SQLite database
- **Automation Sandbox**: AppleScript/automation permissions for sending
- **Privacy Controls**: macOS privacy settings may block access

### Technical Limitations
- **Database Locking**: Messages.app database may be locked during access
- **Real-time Sync**: Live message monitoring may miss rapid message bursts
- **Attachment Handling**: Large media files may cause processing delays
- **Service Detection**: Automatic service selection may choose suboptimal method

## Edge Cases

### Permission and Access Issues
- **Full Disk Access Denied**: System blocking database access
- **Messages.app Offline**: Target application not running or signed out
- **Automation Blocked**: Security settings preventing message sending
- **Database Corruption**: Messages.app database integrity issues
- **Permission Changes**: System revoking previously granted permissions

### Network and Service Problems
- **iMessage Outage**: Apple iMessage service temporarily unavailable
- **SMS Relay Failure**: iPhone connectivity issues preventing SMS delivery
- **Service Confusion**: Messages sent via wrong service (iMessage vs SMS)
- **Delivery Failures**: Messages failing to deliver without clear error indication
- **Account Switching**: Multiple Apple IDs causing service confusion

### Data and Content Issues
- **Large Attachments**: Oversized files failing to send or receive
- **Unsupported Formats**: File types not supported by Messages.app
- **Character Encoding**: Special characters causing message corruption
- **Group Chat Complexity**: Large group conversations affecting performance
- **Message Threading**: Complex reply chains causing display issues

### Real-time Monitoring Complications
- **Watch Process Hanging**: Long-running watch commands becoming unresponsive
- **High Message Volume**: Rapid message flow overwhelming processing
- **Database Changes**: Messages.app schema updates breaking compatibility
- **Resource Exhaustion**: Watch processes consuming excessive system resources
- **Concurrent Access**: Multiple processes accessing Messages database simultaneously

## Technical Debt

### Database Access Fragility
- Direct SQLite database access vulnerable to Messages.app schema changes
- No official API for Messages.app integration
- Potential data corruption risks from concurrent database access
- Limited error recovery for database lock situations

### Permission Management Complexity
- Multiple system permission requirements create setup complexity
- No automated permission validation and setup guidance
- Inconsistent behavior across different macOS versions
- Limited troubleshooting for permission-related failures

### Real-time Processing Limitations
- Basic polling-based approach for message monitoring
- No efficient change notification system for new messages
- Limited scalability for monitoring multiple conversations
- Resource-intensive for long-running watch operations

### Error Handling Gaps
- Generic error messages for diverse failure modes
- Limited context for service-specific failures
- No automatic retry mechanisms for transient issues
- Poor diagnostic information for troubleshooting

## Dependencies

### Core Dependencies
- **imsg Binary**: Primary CLI tool for Messages.app integration
- **Messages.app**: macOS native messaging application
- **SQLite**: Database access for Messages.app data store

### System Dependencies
- **macOS**: Operating system platform requirement (Darwin only)
- **Full Disk Access**: System permission for database access
- **Automation Permissions**: AppleScript/automation framework access
- **Security Framework**: macOS privacy and permission management

### Service Dependencies
- **Apple ID**: Authentication for iMessage service
- **iMessage Service**: Apple's messaging infrastructure
- **SMS Service**: Cellular network or iPhone relay for SMS
- **Internet Connectivity**: Network access for iMessage delivery

### Development Dependencies
- **Homebrew**: Package manager for installation (steipete/tap)
- **Swift/Objective-C**: Implementation language and frameworks
- **AppleScript Bridge**: Automation interface to Messages.app
- **SQLite Libraries**: Database access and query processing

### Optional Dependencies
- **Contacts.app**: Contact information and display names
- **iPhone**: Required for SMS relay functionality
- **Handoff**: Cross-device message continuity
- **Notification Center**: System notification integration