# WACLI Skill Specification

## Intent

Send WhatsApp messages and manage WhatsApp communication through the wacli CLI tool for automated messaging and chat history management (not for normal user conversations).

## Interface

### Core Operations
- **Message Sending**: Send WhatsApp messages to contacts or groups
- **History Search**: Search and retrieve WhatsApp chat history
- **Contact Management**: Manage WhatsApp contacts and groups
- **Message Sync**: Synchronize WhatsApp messages and metadata

### CLI Structure
- **Send Messages**: `wacli send --contact "Name" --message "Text"`
- **Search History**: `wacli search --query "keyword" --contact "Name"`
- **Sync Messages**: `wacli sync --from "2024-01-01" --to "2024-12-31"`
- **List Contacts**: `wacli contacts --list --type groups`

## Business Logic

### Message Delivery and Management
1. Authenticate with WhatsApp Web or API services
2. Send text messages, media, and formatted content
3. Handle message delivery status and read receipts
4. Manage group messaging and broadcast lists

### History and Data Synchronization
1. Download and index WhatsApp chat history
2. Search messages by content, contact, and date ranges
3. Export chat data and conversation archives
4. Handle message encryption and data privacy

### Contact and Group Management
1. Retrieve and organize contact lists and group memberships
2. Handle contact synchronization and updates
3. Manage group administration and member operations
4. Process contact profile information and metadata

## Data Structures

### WhatsApp Contact
```typescript
interface WAContact {
  id: string; // contact identifier
  name: string; // display name
  phone: string; // phone number
  profilePic?: string; // profile picture URL
  status?: string; // status message
  lastSeen?: Date; // last online timestamp
  isGroup: boolean; // group vs individual contact
}
```

### WhatsApp Message
```typescript
interface WAMessage {
  id: string; // message identifier
  chatId: string; // conversation identifier
  fromMe: boolean; // sent by user flag
  timestamp: Date; // message timestamp
  content: MessageContent; // message content
  status: 'sent' | 'delivered' | 'read'; // delivery status
  type: 'text' | 'image' | 'audio' | 'video' | 'document';
}

interface MessageContent {
  text?: string; // text content
  mediaUrl?: string; // media file URL
  caption?: string; // media caption
  filename?: string; // file name
}
```

### Search Query
```typescript
interface SearchQuery {
  query: string; // search text
  contact?: string; // specific contact filter
  dateRange?: { start: Date; end: Date }; // time filter
  messageType?: string; // message type filter
  limit?: number; // result limit
}
```

## Constraints

### WhatsApp Integration Requirements
- **WhatsApp Account**: Valid WhatsApp account required
- **Authentication**: WhatsApp Web or Business API access
- **Rate Limiting**: Message sending frequency restrictions
- **Terms of Service**: Compliance with WhatsApp usage policies

### Privacy and Security Considerations
- **End-to-End Encryption**: Message encryption handling
- **Data Privacy**: User consent for message access and storage
- **Authentication Security**: Secure credential management
- **Compliance**: Regulatory compliance for message archival

### Technical Limitations
- **API Restrictions**: Limited by WhatsApp's API capabilities
- **Message Limits**: Daily message sending quotas
- **Media Support**: File size and format restrictions
- **Real-time Sync**: Message synchronization delays

## Edge Cases

### Authentication and Access Issues
- **Login Failures**: WhatsApp authentication problems
- **Session Expiry**: WhatsApp Web session timeouts
- **Account Suspension**: WhatsApp account restrictions
- **Two-Factor Auth**: Additional security requirements

### Message Delivery Problems
- **Contact Unavailable**: Recipient not reachable or blocked
- **Network Issues**: Internet connectivity affecting delivery
- **Rate Limiting**: Too many messages sent rapidly
- **Content Restrictions**: Blocked or filtered message content

### Data Synchronization Issues
- **Incomplete Sync**: Missing messages during synchronization
- **Duplicate Messages**: Message duplication during sync
- **Encryption Keys**: Inability to decrypt message content
- **Storage Limits**: Local storage constraints for message history

## Technical Debt

### Authentication Complexity
- Complex WhatsApp authentication flow management
- Limited error handling for authentication failures
- No automatic re-authentication mechanisms

### Data Management
- Basic message storage without advanced indexing
- Limited message format standardization
- No conflict resolution for synchronized data

### Integration Limitations
- Command-line only interface without programmatic API
- Limited real-time event handling capabilities
- Basic error reporting without detailed diagnostics

## Dependencies

### Core Dependencies
- **wacli**: WhatsApp CLI tool
- **WhatsApp Account**: Valid WhatsApp user account
- **Network Connectivity**: Internet access for WhatsApp services
- **Authentication**: WhatsApp Web or API credentials

### System Dependencies
- **Web Browser**: Browser automation for WhatsApp Web (if used)
- **Storage**: Local storage for message history and media
- **Encryption**: Cryptographic libraries for message handling
- **JSON Processing**: Message data parsing and manipulation

### Optional Dependencies
- **Database**: Structured storage for message indexing
- **Media Processing**: Image and audio processing capabilities
- **Backup Tools**: Message backup and restore functionality
- **Integration APIs**: Connection to other messaging or CRM systems