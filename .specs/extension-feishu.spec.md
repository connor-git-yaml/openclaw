# Extension Specification: Feishu

## Intent

Feishu extension provides comprehensive integration with Feishu/Lark enterprise messaging platform, enabling both communication and productivity workflows. It serves as a channel plugin for messaging capabilities while offering extensive toolsets for document management, knowledge base operations, file storage, permission handling, and database interactions through Feishu's Bitable feature. The extension bridges OpenClaw's capabilities with Feishu's enterprise ecosystem.

## Interface

### Plugin Registration
- **ID**: `feishu`
- **Name**: Feishu
- **Type**: Channel Plugin + Tool Collection
- **Entry Point**: `index.ts` exports plugin definition with multi-component registration
- **API Surface**: Channel plugin, document tools, wiki tools, drive tools, permission tools, bitable tools

### Channel Plugin Methods
- `sendMessage()` - Send text/media messages to individuals or groups
- `sendCard()` - Send rich interactive card messages
- `updateCard()` - Update existing card messages
- `editMessage()` - Edit previously sent messages
- `addReaction()` - Add emoji reactions to messages
- `uploadMedia()` - Upload and share files, images, videos

### Tool Collections
- **Document Tools**: Create, edit, manage Feishu documents
- **Wiki Tools**: Knowledge base content management and organization
- **Drive Tools**: File storage, sharing, and collaboration operations
- **Permission Tools**: User and group access control management
- **Bitable Tools**: Database operations, records, and data management

### Export Functions
```typescript
// Core messaging functions
sendMessageFeishu, sendCardFeishu, updateCardFeishu
editMessageFeishu, getMessageFeishu

// Media handling
uploadImageFeishu, uploadFileFeishu, sendImageFeishu
sendFileFeishu, sendMediaFeishu

// Reactions and mentions
addReactionFeishu, removeReactionFeishu, listReactionsFeishu
extractMentionTargets, buildMentionedMessage
```

## Business Logic

### Multi-Component Architecture
- **Channel Integration**: Primary messaging interface with Feishu conversations
- **Productivity Tools**: Document, wiki, and file management capabilities
- **Permission System**: Enterprise-grade access control and user management
- **Database Interface**: Bitable integration for structured data operations

### Authentication & Connection
- **App Authentication**: Feishu app credentials and token management
- **User Authentication**: Individual user token handling for personal operations
- **Webhook Processing**: Real-time message and event receiving
- **API Rate Limiting**: Compliance with Feishu's API rate limits

### Message Processing
- **Rich Content**: Support for cards, mentions, reactions, and media attachments
- **Event Handling**: Processes incoming messages, mentions, reactions, and system events
- **Target Resolution**: Maps user IDs, group IDs to conversation contexts
- **Mention System**: @user and @all mention parsing and formatting

### Enterprise Features
- **Directory Integration**: User and group discovery through Feishu directory
- **Permission Management**: Role-based access control for various resources
- **Document Collaboration**: Real-time document editing and sharing workflows
- **Knowledge Management**: Wiki creation, organization, and content management

## Data Structures

### Account Configuration
```typescript
FeishuAccountConfig {
  name?: string;
  appId?: string;
  appSecret?: string;
  webhookPath?: string;
  botToken?: string;
  capabilities?: string[];
  dmPolicy?: DmPolicy;
  groupPolicy?: GroupPolicy;
  allowFrom?: string[];
}
```

### Message Types
```typescript
FeishuMessage {
  messageId: string;
  chatId: string;
  userId: string;
  content: FeishuContent;
  mentions?: MentionTarget[];
  timestamp: number;
}

FeishuCard {
  cardId: string;
  template: string;
  variables: Record<string, any>;
  actions?: CardAction[];
}
```

### Tool Integration
```typescript
DocumentTool {
  docId: string;
  title: string;
  type: "docx" | "sheet" | "bitable";
  permissions: Permission[];
  collaborators: User[];
}

BitableRecord {
  tableId: string;
  recordId: string;
  fields: Record<string, any>;
  metadata: RecordMetadata;
}
```

## Constraints

### Technical Limitations
- **API Rate Limits**: Feishu enforces strict rate limiting on API calls
- **Webhook Dependencies**: Requires accessible webhook endpoint for real-time features
- **Token Expiration**: App and user tokens require periodic refresh
- **File Size Limits**: Media uploads constrained by Feishu's size restrictions

### Enterprise Constraints
- **Permission Model**: Complex enterprise permission system affects tool access
- **Tenant Isolation**: Multi-tenant deployments require careful account isolation
- **Compliance Requirements**: Enterprise data handling and retention policies
- **Network Access**: May require VPN or specific network configurations

### Feature Limitations
- **Platform Differences**: Feishu vs Lark feature parity variations
- **Card Interactions**: Limited interactive card capabilities compared to native apps
- **Document Editing**: Concurrent editing limitations and conflict resolution
- **Search Scope**: Limited search capabilities across different content types

## Edge Cases

### Authentication Issues
- **Token Refresh Failures**: Expired tokens causing operation failures
- **Permission Denied**: Insufficient permissions for requested operations
- **App Configuration**: Misconfigured app settings blocking functionality
- **Multi-Tenant Conflicts**: Account isolation failures in shared environments

### Message Handling
- **Large Group Messages**: Performance issues with very large group conversations
- **Rich Content Limits**: Complex cards exceeding platform capabilities
- **Mention Resolution**: Ambiguous user mentions in large organizations
- **Message Threading**: Complex conversation thread handling

### Tool Integration
- **Document Conflicts**: Simultaneous editing creating content conflicts
- **Permission Inheritance**: Complex permission cascading in nested structures
- **File Synchronization**: Version control issues with shared documents
- **Database Constraints**: Bitable schema limitations and data validation

### Network and Connectivity
- **Webhook Reliability**: Missed events during network outages
- **API Endpoint Changes**: Feishu service updates affecting integration
- **Cross-Region Access**: Geographic restrictions on API access
- **Firewall Issues**: Enterprise firewall blocking webhook delivery

## Technical Debt

### Code Organization
- Large codebase with multiple responsibilities mixed in single modules
- Inconsistent error handling patterns across different tool collections
- Duplicate authentication logic between channel and tool components
- Missing abstraction layers for common Feishu API operations

### Performance Issues
- Synchronous API calls could benefit from batching and async processing
- No caching layer for frequently accessed directory and permission data
- Inefficient polling mechanisms for real-time updates
- Memory leaks in long-running webhook processors

### Testing and Maintenance
- Limited integration tests for complex enterprise workflow scenarios
- Missing mock frameworks for Feishu API testing
- Insufficient error simulation for network and permission failures
- Manual testing requirements for enterprise feature validation

### Documentation and Monitoring
- Inadequate logging for debugging complex permission and workflow issues
- Missing metrics for API usage, rate limiting, and performance monitoring
- Limited documentation for enterprise configuration and deployment
- No automated health checks for critical integration components

## Dependencies

### External Dependencies
- **Feishu/Lark Platform**: Core messaging and productivity services
- **Feishu Open Platform**: APIs for messaging, documents, and enterprise features
- **Webhook Infrastructure**: Real-time event delivery system
- **Enterprise Directory**: User and group management services

### Internal Dependencies
- **openclaw/plugin-sdk**: Core plugin framework and channel infrastructure
- **HTTP Client**: API communication and webhook processing
- **Authentication System**: Token management and credential storage
- **File System**: Media upload and document processing capabilities

### Runtime Requirements
- **Network Connectivity**: Stable connection to Feishu APIs and webhook delivery
- **Storage**: Document cache, media uploads, and configuration persistence
- **Memory**: Sufficient memory for large document and media processing
- **CPU**: Processing overhead for rich content rendering and API operations