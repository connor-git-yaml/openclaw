# Extension Specification: iMessage

## Intent

iMessage extension provides direct integration with Apple's iMessage service on macOS systems, enabling OpenClaw to send and receive messages through the native Messages app. It leverages macOS system APIs and AppleScript automation to interact with the Messages app, providing seamless messaging capabilities while respecting Apple's ecosystem constraints and user privacy requirements.

## Interface

### Plugin Registration
- **ID**: `imessage`
- **Name**: iMessage
- **Type**: Channel Plugin
- **Entry Point**: `index.ts` exports plugin definition
- **API Surface**: Channel plugin with native macOS integration

### Channel Plugin Methods
- `sendMessage()` - Send text messages to phone numbers or email addresses
- `sendMedia()` - Send images, videos, and file attachments
- `getMessages()` - Retrieve message history from Messages app database
- `markAsRead()` - Mark conversations as read in Messages app
- `getContacts()` - Access contact information from system address book

### Target Support
- **Phone Numbers**: International and domestic phone number formats
- **Email Addresses**: iMessage-enabled email addresses
- **Groups**: Existing group conversations in Messages app
- **Aliases**: Contact names and system address book integration

### Native Integration
- **Messages App**: Direct interaction with macOS Messages application
- **AppleScript**: Automation scripts for message operations
- **SQLite Database**: Direct access to Messages app database for history
- **System Contacts**: Integration with macOS Contacts app

## Business Logic

### Platform Requirements
- **macOS Only**: Strictly limited to macOS systems with Messages app
- **iMessage Account**: Requires signed-in Apple ID with iMessage enabled
- **Accessibility Permissions**: System permissions for app automation
- **Messages App State**: Messages app must be available (not necessarily running)

### Message Processing
- **Outbound Delivery**: Send messages through Messages app automation
- **Inbound Monitoring**: Poll Messages database for new incoming messages
- **Contact Resolution**: Map phone numbers/emails to contact names
- **Group Handling**: Support for existing group conversations

### Database Integration
- **SQLite Access**: Direct read access to Messages app database
- **Message History**: Retrieve conversation history and metadata
- **Contact Mapping**: Link messages to contact information
- **Thread Management**: Handle conversation threading and organization

### Automation Workflow
- **AppleScript Execution**: Automate Messages app through system scripting
- **Permission Handling**: Request and manage system automation permissions
- **App State Management**: Handle Messages app activation and focus
- **Error Recovery**: Graceful handling of automation failures

## Data Structures

### Message Format
```typescript
IMessageData {
  guid: string; // Unique message identifier
  text?: string; // Message content
  handle: string; // Sender phone/email
  service: "iMessage" | "SMS"; // Message service type
  date: number; // Timestamp
  isFromMe: boolean; // Sent vs received
  chatGuid?: string; // Conversation identifier
}
```

### Contact Information
```typescript
ContactInfo {
  id: string; // Contact identifier
  displayName?: string; // Contact name
  handles: string[]; // Phone numbers and emails
  isContact: boolean; // In address book
}
```

### Target Resolution
```typescript
IMessageTarget {
  handle: string; // Phone number or email
  chatGuid?: string; // Group conversation ID
  displayName?: string; // Resolved contact name
  service: "iMessage" | "SMS"; // Delivery service
}
```

### Configuration Schema
```typescript
IMessageConfig {
  enabled?: boolean;
  pollInterval?: number; // Database polling frequency
  maxHistory?: number; // Message history limit
  contactSync?: boolean; // Address book integration
}
```

## Constraints

### Platform Limitations
- **macOS Exclusive**: Cannot function on other operating systems
- **Messages App Dependency**: Requires installed and configured Messages app
- **Apple ID Requirement**: Must be signed into iMessage service
- **Network Connectivity**: Requires internet for iMessage delivery

### System Permissions
- **Accessibility Access**: Automation permissions for Messages app control
- **Full Disk Access**: Database access permissions for message history
- **Privacy Permissions**: User consent for contact and message access
- **App Permissions**: Messages app must allow external automation

### Technical Constraints
- **Database Locking**: Messages app database may be locked during use
- **AppleScript Limitations**: Script execution timeouts and failures
- **Message Ordering**: Race conditions in rapid message sequences
- **Media Handling**: Limited file type support for attachments

### Service Limitations
- **iMessage vs SMS**: Service availability depends on recipient capabilities
- **Group Restrictions**: Limited control over group conversation membership
- **Message Editing**: No support for message modification after sending
- **Read Receipts**: Limited control over read receipt behavior

## Edge Cases

### Permission Issues
- **Denied Permissions**: User denying required system permissions
- **Permission Revocation**: Permissions removed after initial grant
- **Security Updates**: macOS updates affecting permission requirements
- **App Sandboxing**: Restrictions from app sandbox policies

### Messages App States
- **App Not Running**: Messages app not launched when needed
- **App Crashes**: Messages app instability affecting automation
- **Database Corruption**: Messages database integrity issues
- **Account Issues**: iMessage account sign-out or authentication problems

### Network and Connectivity
- **Offline Mode**: No internet connectivity for iMessage delivery
- **Service Outages**: Apple iMessage service interruptions
- **Account Sync Issues**: iMessage account synchronization problems
- **Cross-Device Conflicts**: Message delivery conflicts across multiple devices

### Data Consistency
- **Database Changes**: Messages app database schema changes in macOS updates
- **Message Sync Delays**: Delayed message appearance in local database
- **Contact Updates**: Address book changes not reflected in message handling
- **Duplicate Messages**: Multiple delivery attempts causing duplicates

## Technical Debt

### Security and Privacy
- **Database Access**: Direct SQLite access bypasses Apple's privacy controls
- **Permission Management**: Manual permission setup creates user friction
- **Credential Storage**: No secure credential storage for automation
- **Audit Trail**: Limited logging for message access and operations

### Code Organization
- **AppleScript Embedding**: Inline scripts make testing and maintenance difficult
- **Database Queries**: Raw SQL queries scattered throughout codebase
- **Error Handling**: Inconsistent error handling for system automation failures
- **Platform Abstractions**: Tight coupling to macOS-specific APIs

### Performance Issues
- **Database Polling**: Inefficient polling for new messages
- **Script Execution**: Slow AppleScript execution affecting responsiveness
- **Large History**: Performance degradation with large message histories
- **Memory Usage**: Potential memory leaks from database connections

### Maintenance Challenges
- **macOS Compatibility**: Brittleness across macOS version updates
- **Messages App Changes**: Apple Messages app updates breaking automation
- **Testing Limitations**: Difficult to automate testing of system integration
- **Documentation**: Limited documentation for setup and troubleshooting

## Dependencies

### External Dependencies
- **macOS Messages App**: Core messaging functionality and database
- **Apple iMessage Service**: Remote messaging service infrastructure
- **System Contacts**: macOS address book for contact resolution
- **AppleScript Runtime**: System automation scripting environment

### Internal Dependencies
- **openclaw/plugin-sdk**: Core channel framework and message processing
- **Database Access**: SQLite drivers for Messages database access
- **File System**: Access to Messages app data directories
- **Process Management**: Automation of external applications

### Runtime Requirements
- **macOS Operating System**: Minimum supported macOS version compatibility
- **System Permissions**: Accessibility, Full Disk Access, and Privacy permissions
- **Disk Space**: Storage for message history caching and temporary files
- **Network Access**: Internet connectivity for iMessage service communication