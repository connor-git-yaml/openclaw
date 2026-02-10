# Messaging Module Specification

## Intent
The Messaging module provides the core communication infrastructure for OpenClaw agents, enabling cross-channel message delivery, channel management, and message processing workflows. It abstracts multiple messaging platforms (Discord, Telegram, Slack, iMessage, etc.) behind a unified interface, supporting rich media, reactions, threads, and advanced messaging features across different platforms.

## Interface

### Core Types
```typescript
export type MessageAction = 'send' | 'edit' | 'delete' | 'pin' | 'unpin' | 'react' | 
  'thread-create' | 'thread-reply' | 'poll' | 'sticker' | 'upload' | 'broadcast';

export type MessageRequest = {
  action: MessageAction;
  target: string;
  message?: string;
  media?: string;
  buffer?: string;
  filename?: string;
  contentType?: string;
  replyTo?: string;
  threadId?: string;
  targets?: string[];
  emoji?: string;
  effect?: string;
  silent?: boolean;
  timeoutMs?: number;
  gatewayUrl?: string;
  gatewayToken?: string;
};

export type MessageResult = {
  success: boolean;
  messageId?: string;
  channelId?: string;
  timestamp: Date;
  provider: string;
  deliveryStatus: DeliveryStatus;
  error?: string;
};

export type Channel = {
  id: string;
  name: string;
  type: 'dm' | 'group' | 'channel' | 'thread';
  provider: string;
  participants: Participant[];
  capabilities: ChannelCapability[];
  metadata: Record<string, any>;
};

export type DeliveryStatus = 'sent' | 'delivered' | 'read' | 'failed' | 'queued';
```

### Key Functions
- `sendMessage(channel, content, options)` - Send message to channel
- `editMessage(messageId, newContent)` - Edit existing message
- `deleteMessage(messageId)` - Delete message
- `reactToMessage(messageId, emoji)` - Add reaction to message
- `createThread(parentMessage, title)` - Create message thread
- `broadcastMessage(channels, content)` - Send to multiple channels
- `uploadMedia(channel, media, caption)` - Upload media with message

### Platform Support
- **Discord**: Full bot functionality with embeds, reactions, threads
- **Telegram**: Bot API with media, inline keyboards, webhooks
- **Slack**: App platform with blocks, workflows, reactions
- **iMessage**: Native macOS integration with media support
- **Signal**: End-to-end encrypted messaging
- **WhatsApp**: Business API integration
- **Matrix**: Decentralized messaging protocol

## Business Logic

### Message Routing and Delivery
1. **Channel Resolution**: Resolve target channel from identifier
2. **Provider Selection**: Determine appropriate messaging provider
3. **Message Formatting**: Format message for target platform
4. **Media Processing**: Process and optimize media attachments
5. **Delivery Execution**: Send message via provider API
6. **Status Tracking**: Track delivery and read status
7. **Error Handling**: Handle delivery failures with retry logic
8. **Confirmation**: Provide delivery confirmation to sender

### Cross-Platform Message Translation
- **Format Adaptation**: Adapt message format for different platforms
- **Feature Mapping**: Map features across platform capabilities
- **Media Conversion**: Convert media formats for platform compatibility
- **Markup Translation**: Convert between different markup languages
- **Link Handling**: Platform-specific link preview and embedding

### Rich Media Handling
- **File Upload**: Support various file types and size limits
- **Image Processing**: Automatic image optimization and resizing
- **Video Handling**: Video compression and format conversion
- **Audio Support**: Voice messages and audio file handling
- **Document Support**: PDF, Office docs, and other document types

### Thread and Conversation Management
- **Thread Creation**: Create threaded conversations where supported
- **Reply Handling**: Maintain reply context across platforms
- **Conversation State**: Track conversation state and participants
- **Message Ordering**: Ensure correct message ordering in threads
- **Notification Management**: Control thread notification settings

### Reaction and Interaction System
- **Emoji Reactions**: Cross-platform emoji reaction support
- **Custom Reactions**: Platform-specific custom reaction handling
- **Reaction Aggregation**: Collect and display reaction counts
- **Interaction Callbacks**: Handle interactive message responses
- **Button and Menu Support**: Interactive UI elements where supported

## Data Structures

### Message Queue Entry
```typescript
type MessageQueueEntry = {
  id: string;
  channel: string;
  content: string;
  media?: MediaAttachment[];
  provider: string;
  priority: number;
  retryCount: number;
  scheduledTime: Date;
  timeout: number;
  status: 'queued' | 'sending' | 'sent' | 'failed';
};
```

### Channel Capability
```typescript
type ChannelCapability = 'text' | 'media' | 'reactions' | 'threads' | 
  'editing' | 'deletion' | 'polls' | 'stickers' | 'effects' | 'voice';
```

### Media Attachment
```typescript
type MediaAttachment = {
  type: 'image' | 'video' | 'audio' | 'document';
  url?: string;
  buffer?: Buffer;
  filename: string;
  size: number;
  mimeType: string;
  caption?: string;
  thumbnail?: string;
};
```

### Delivery Receipt
```typescript
type DeliveryReceipt = {
  messageId: string;
  channelId: string;
  status: DeliveryStatus;
  timestamp: Date;
  readBy?: string[];
  deliveredTo?: string[];
  failureReason?: string;
};
```

### Channel State
```typescript
type ChannelState = {
  id: string;
  provider: string;
  lastActivity: Date;
  messageCount: number;
  participants: Map<string, Participant>;
  permissions: ChannelPermissions;
  settings: ChannelSettings;
  metadata: Record<string, any>;
};
```

## Constraints

### Platform-Specific Limitations
- **Message Length**: Varying maximum message length across platforms
- **Media Size**: Different file size limits for media uploads
- **Rate Limits**: Platform-specific rate limiting and throttling
- **Feature Support**: Not all features available on all platforms

### Delivery Constraints
- **Timeout Limits**: Maximum delivery attempt timeouts
- **Retry Limits**: Maximum retry attempts for failed messages
- **Queue Size**: Maximum pending message queue size
- **Concurrent Delivery**: Limits on concurrent message sending

### Content Restrictions
- **File Types**: Platform-specific allowed file types
- **Content Filtering**: Automatic content moderation and filtering
- **Privacy Controls**: Respect user privacy and blocking settings
- **Regional Restrictions**: Comply with regional content regulations

### Performance Constraints
- **Processing Time**: Target message processing under 3 seconds
- **Memory Usage**: Message processing limited to 100MB memory
- **Storage**: Message cache and media storage limitations
- **Bandwidth**: Optimize for low-bandwidth environments

## Edge Cases

### Delivery Failure Scenarios
- **Network Outages**: Handle provider API unavailability
- **Rate Limiting**: Graceful handling when rate limits exceeded
- **Authentication Issues**: Handle expired tokens and auth failures
- **Channel Unavailability**: Handle deleted or inaccessible channels

### Cross-Platform Compatibility Issues
- **Feature Mismatches**: Handle features not supported on target platform
- **Format Incompatibility**: Convert incompatible media formats
- **Character Limits**: Handle messages exceeding platform limits
- **Encoding Issues**: Handle various text encoding problems

### Message State Synchronization
- **Duplicate Messages**: Prevent duplicate message delivery
- **Message Ordering**: Maintain correct order in high-volume channels
- **State Conflicts**: Resolve conflicts between different message states
- **Recovery After Outage**: Proper state recovery after system restart

### User Interaction Edge Cases
- **Deleted Users**: Handle messages to/from deleted user accounts
- **Blocked Relationships**: Respect user blocking and privacy settings
- **Permission Changes**: Handle changing channel permissions
- **Channel Migration**: Handle channel moves and reorganization

## Technical Debt

### Known Issues
- **Error Reporting**: Inconsistent error messages across providers
- **State Persistence**: Message queue state not persisted across restarts
- **Rate Limit Handling**: Basic rate limiting without intelligent backoff
- **Media Processing**: Limited media optimization and format conversion

### Performance Optimizations Needed
- **Message Batching**: Batch multiple messages for efficiency
- **Connection Pooling**: Reuse connections across message sends
- **Cache Optimization**: Smarter caching of channel and user data
- **Media Pipeline**: More efficient media processing pipeline

### Feature Gaps
- **Message Scheduling**: No support for delayed message sending
- **Message Templates**: No templating system for common message types
- **Analytics**: Limited messaging analytics and performance metrics
- **Webhook Integration**: Limited webhook support for message events

### Architecture Improvements Required
- **Plugin System**: More extensible plugin system for new providers
- **Event System**: Better event handling for message lifecycle
- **State Management**: More robust distributed state management
- **Load Balancing**: Distribute load across multiple provider instances

## Dependencies

### Internal Dependencies
- **Gateway**: WebSocket communication for message coordination
- **Config**: Channel configuration and provider credentials
- **Security**: Message encryption and content filtering
- **Tools**: Integration with OpenClaw tool system

### External Dependencies
- **Provider APIs**: Discord, Telegram, Slack, etc. APIs
- **Media Processing**: FFmpeg for media conversion
- **HTTP Client**: HTTP library for provider API requests
- **WebSocket**: Real-time communication with messaging platforms

### Platform Dependencies
- **macOS**: iMessage integration via AppleScript/Shortcuts
- **Linux**: DBus for desktop notification integration
- **Windows**: Windows notification system integration
- **Mobile**: Platform-specific push notification systems