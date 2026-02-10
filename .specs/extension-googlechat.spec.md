# Extension Specification: Google Chat

## Intent

Google Chat extension enables OpenClaw integration with Google Chat (formerly Hangouts Chat), providing enterprise messaging capabilities through Google Workspace. It functions as a channel plugin supporting both 1:1 direct messages and group spaces, with comprehensive message handling, file attachments, and webhook-based real-time communication. The extension facilitates seamless AI assistant integration within Google's enterprise communication ecosystem.

## Interface

### Plugin Registration
- **ID**: `googlechat`
- **Name**: Google Chat
- **Type**: Channel Plugin
- **Entry Point**: `index.ts` exports plugin definition
- **API Surface**: Channel plugin, webhook handler, and dock integration

### Channel Plugin Methods
- `sendMessage()` - Send text messages to users or spaces
- `uploadAttachment()` - Upload and share files with size limitations
- `editMessage()` - Modify previously sent messages (if supported)
- `deleteMessage()` - Remove messages from conversations
- `addReaction()` - Add emoji reactions to messages

### HTTP Endpoints
- **Webhook Handler**: Receives real-time events from Google Chat
- **Path Configuration**: Customizable webhook paths per account
- **Event Processing**: Handles message, mention, and space events

### Dock Integration
- **Space Management**: Interface for joining and managing Google Chat spaces
- **User Discovery**: Finding and connecting with Google Chat users
- **Configuration UI**: Account setup and preference management

## Business Logic

### Authentication & Authorization
- **Service Account**: Google Cloud service account credentials for API access
- **Webhook Authentication**: Request signature verification for incoming events
- **Scope Management**: Appropriate Google Chat API scopes for bot functionality
- **Token Refresh**: Automatic JWT token generation and renewal

### Message Processing Pipeline
- **Outbound Messages**: Format and send messages through Google Chat API
- **Inbound Events**: Process webhooks for real-time message reception
- **Target Resolution**: Map user emails and space names to Google Chat IDs
- **Content Formatting**: Rich text, mentions, and attachment handling

### Space Management
- **Space Discovery**: Finding and joining Google Chat spaces
- **Membership Management**: Handling space member additions and removals
- **Permission Handling**: Respecting space-level permissions and policies
- **Thread Management**: Supporting threaded conversations when available

### Policy Enforcement
- **DM Policy**: Controls for direct message access and responses
- **Space Policy**: Group conversation participation rules
- **Mention Requirements**: Configurable mention-only response modes
- **File Upload Limits**: Size and type restrictions for attachments

## Data Structures

### Account Configuration
```typescript
GoogleChatAccountConfig {
  name?: string;
  serviceAccountKey?: string; // JSON key file content
  projectId?: string;
  webhookPath?: string;
  capabilities?: string[];
  dmPolicy?: "pairing" | "allowlist" | "open" | "disabled";
  spacePolicy?: "open" | "disabled" | "allowlist";
  allowFrom?: string[];
}
```

### Message Types
```typescript
GoogleChatMessage {
  messageId: string;
  spaceId: string;
  senderId: string;
  text?: string;
  cards?: Card[];
  attachments?: Attachment[];
  thread?: ThreadInfo;
  createTime: string;
}
```

### Space Information
```typescript
GoogleChatSpace {
  name: string; // spaces/{spaceId}
  displayName: string;
  type: "ROOM" | "DM" | "GROUP_CHAT";
  spaceType: "SPACE" | "GROUP_CHAT" | "DIRECT_MESSAGE";
  members: Member[];
  threaded: boolean;
}
```

### Webhook Events
```typescript
WebhookEvent {
  type: "MESSAGE" | "ADDED_TO_SPACE" | "REMOVED_FROM_SPACE";
  eventTime: string;
  message?: GoogleChatMessage;
  space: GoogleChatSpace;
  user: User;
}
```

## Constraints

### API Limitations
- **Rate Limits**: Google Chat API enforces per-minute request quotas
- **Message Size**: Text content limited to specific character counts
- **File Upload**: Attachment size limits (typically 200MB)
- **API Scopes**: Restricted functionality based on configured scopes

### Authentication Constraints
- **Service Account**: Requires Google Cloud project and service account setup
- **Domain Restrictions**: May be limited to specific Google Workspace domains
- **Webhook Security**: Requires HTTPS endpoints for production webhooks
- **Token Lifecycle**: JWT tokens have limited lifespans requiring regeneration

### Feature Limitations
- **Message Editing**: Limited or no support for message modification
- **Thread Support**: Threading capabilities vary by space configuration
- **Rich Formatting**: Limited rich text and formatting options
- **Bot Interactions**: Restrictions on bot-to-bot communications

### Enterprise Restrictions
- **Domain Policies**: Google Workspace admin policies may restrict bot usage
- **Data Residency**: Geographic restrictions on data processing
- **External Users**: Limitations on interactions with external domains
- **Compliance**: GDPR, SOC2, and other compliance requirements

## Edge Cases

### Authentication Issues
- **Service Account Expiry**: Service account keys requiring rotation
- **Permission Changes**: Dynamic permission changes affecting bot access
- **Domain Configuration**: Google Workspace domain settings blocking bot access
- **API Quota Exhaustion**: Rate limits causing temporary service disruption

### Message Handling
- **Large Messages**: Handling messages exceeding size limits
- **Malformed Content**: Processing invalid or corrupted message content
- **Duplicate Events**: Webhook event deduplication and idempotency
- **Out-of-Order Delivery**: Messages arriving in non-chronological order

### Space Management
- **Space Deletion**: Handling space removal while bot is active
- **Permission Changes**: Dynamic permission updates affecting bot capabilities
- **Member Management**: Users joining/leaving spaces during operations
- **Space Type Changes**: Conversion between different space types

### Network and Infrastructure
- **Webhook Delivery**: Failed webhook delivery requiring retry mechanisms
- **API Downtime**: Google Chat service outages and degradation
- **Network Partitions**: Connectivity issues affecting real-time operations
- **Load Balancing**: Webhook distribution across multiple instances

## Technical Debt

### Code Organization
- **Monolithic Modules**: Large modules combining multiple responsibilities
- **Inconsistent Error Handling**: Mixed error handling patterns across components
- **Configuration Complexity**: Complex configuration schema with nested options
- **Duplicate Logic**: Repeated patterns for user and space target resolution

### Performance Issues
- **Synchronous Processing**: Blocking webhook processing affecting throughput
- **Memory Usage**: Large message history and space metadata caching
- **API Efficiency**: Suboptimal API call patterns and batching opportunities
- **Connection Pooling**: Missing connection reuse for API calls

### Testing and Reliability
- **Integration Testing**: Limited real Google Chat API testing
- **Mock Frameworks**: Insufficient mocking for complex Google Chat scenarios
- **Error Simulation**: Missing chaos testing for webhook and API failures
- **Load Testing**: Unknown performance characteristics under high message volume

### Security and Compliance
- **Credential Management**: Service account key storage and rotation procedures
- **Audit Logging**: Insufficient logging for compliance and security monitoring
- **Data Privacy**: Message content handling and retention policies
- **Access Control**: Granular permission management and validation

## Dependencies

### External Dependencies
- **Google Chat API**: Core messaging and space management services
- **Google Cloud Platform**: Service account authentication and project infrastructure
- **Google Workspace**: Enterprise directory and user management
- **Webhook Infrastructure**: Real-time event delivery system

### Internal Dependencies
- **openclaw/plugin-sdk**: Core channel framework and message processing
- **HTTP Server**: Webhook endpoint hosting and request processing
- **Authentication System**: JWT generation and service account management
- **Configuration Management**: Account setup and policy enforcement

### Runtime Requirements
- **Network Connectivity**: Stable connection to Google Chat APIs
- **HTTPS Support**: SSL/TLS for webhook endpoints (production)
- **Storage**: Configuration, credentials, and message history persistence
- **Memory**: Sufficient memory for message processing and space metadata caching