# Discord Skill Specification

## Intent

Provide comprehensive Discord integration for OpenClaw through bot APIs, enabling message management, reactions, media sharing, moderation, and community management. Supports both casual messaging and automated workflows for Discord servers and DMs.

## Interface

### Message Operations
- **send/edit/delete** - Text and media message management
- **react/reactions** - Emoji reactions and reaction listing
- **sticker** - Send official and custom Discord stickers
- **poll** - Create polls with multiple options and duration

### Media and Content
- **emojiUpload/emojiList** - Custom emoji management
- **stickerUpload** - Custom sticker creation
- **mediaUrl** - File attachments (local and remote)

### Channel and Server Management
- **channelCreate/edit/delete** - Channel lifecycle (disabled by default)
- **categoryCreate/edit/delete** - Channel category management
- **channelInfo/channelList** - Channel information retrieval
- **threadCreate/threadList/threadReply** - Thread management

### Moderation and Administration
- **timeout/kick/ban** - User moderation actions (disabled by default)
- **roleAdd/roleInfo** - Role management (disabled by default)
- **permissions** - Bot permission checking
- **setPresence** - Bot status and activity (disabled by default)

## Business Logic

### Message Routing and Delivery
1. Route messages to channels (`channel:<id>`) or direct messages (`user:<id>`)
2. Handle message formatting with Discord markdown and emoji support
3. Process file attachments from local paths or remote URLs
4. Manage message threading and reply contexts

### Permission and Safety System
1. Check bot permissions before attempting operations
2. Implement action gating via `discord.actions.*` configuration
3. Validate user permissions for moderation actions
4. Handle rate limiting and API quota management

### Media Processing Pipeline
1. Validate file types and size limits for uploads
2. Process emoji uploads (PNG/JPG/GIF, ≤256KB)
3. Handle sticker uploads (PNG/APNG/Lottie JSON, ≤512KB)
4. Manage attachment URLs and file serving

### Community Interaction Workflows
1. Create and manage polls with time limits and multi-select options
2. Handle emoji reactions and reaction aggregation
3. Manage threads for structured discussions
4. Coordinate server events and announcements

## Data Structures

### Discord Message
```typescript
interface DiscordMessage {
  id: string;                  // message identifier
  channelId: string;           // channel containing message
  guildId?: string;            // server identifier (null for DMs)
  content: string;             // message text content
  author: DiscordUser;         // message author
  timestamp: Date;             // message creation time
  editedTimestamp?: Date;      // last edit time
  attachments: DiscordAttachment[];
  reactions: DiscordReaction[];
  referencedMessage?: string;  // replied-to message ID
}
```

### Discord Channel
```typescript
interface DiscordChannel {
  id: string;                  // channel identifier
  type: number;                // channel type (0=text, 2=voice, 4=category)
  guildId?: string;            // server identifier
  name: string;                // channel name
  topic?: string;              // channel topic/description
  parentId?: string;           // parent category ID
  position: number;            // display position
  nsfw: boolean;               // age-restricted content flag
  rateLimitPerUser?: number;   // slowmode seconds
}
```

### Discord Poll
```typescript
interface DiscordPoll {
  question: string;            // poll question text
  answers: string[];           // available answer options (2-10)
  allowMultiselect: boolean;   // multiple choice selection
  durationHours: number;       // poll duration (max 768 hours)
  expiresAt: Date;             // calculated expiry timestamp
}
```

### Custom Media Asset
```typescript
interface DiscordAsset {
  id: string;                  // asset identifier
  guildId: string;             // owning server
  name: string;                // asset name
  type: 'emoji' | 'sticker';   // asset type
  url: string;                 // CDN URL
  size: number;                // file size in bytes
  format: string;              // file format (PNG, GIF, etc.)
  roleIds?: string[];          // role restrictions (emoji only)
  tags?: string;               // sticker tags
  description?: string;        // sticker description
}
```

## Constraints

### Discord API Limitations
- **Rate Limits**: Strict API rate limiting per endpoint and operation type
- **File Size**: 8MB attachment limit for regular users, 50MB for Nitro servers
- **Message Length**: 2000 character limit for message content
- **Emoji Limits**: 50 custom emojis for normal servers, 250 for boosted servers

### Bot Permission Requirements
- **Message Operations**: Requires Send Messages, Read Message History
- **Reactions**: Requires Add Reactions permission
- **Moderation**: Requires specific permissions (Timeout Members, Kick Members, etc.)
- **Channel Management**: Requires Manage Channels permission

### Media Upload Restrictions
- **Emoji**: PNG/JPG/GIF formats, 256KB maximum size
- **Stickers**: PNG/APNG/Lottie JSON formats, 512KB maximum size
- **Attachments**: Various formats supported, server-specific size limits
- **Animation**: GIF emoji limited to 3 seconds duration

### Action Gating Defaults
- **Enabled by default**: reactions, stickers, polls, permissions, messages, threads, pins, search
- **Disabled by default**: roles, channels, moderation, presence
- **Safety-first**: Destructive actions require explicit enablement

## Edge Cases

### Permission and Authentication Issues
- **Insufficient Bot Permissions**: Operations failing due to missing permissions
- **User Permission Changes**: Bot permissions revoked during operation
- **Channel Permission Overrides**: Channel-specific permissions blocking actions
- **DM Restrictions**: Certain operations not available in direct messages

### Network and API Failures
- **Rate Limit Exceeded**: API throttling causing operation delays
- **Discord Service Outage**: Discord API temporarily unavailable
- **Webhook Failures**: Webhook endpoints not responding
- **Large Server Latency**: Slow operations in high-activity servers

### Content and Media Problems
- **File Upload Failures**: Attachments exceeding size limits or unsupported formats
- **Invalid URLs**: Remote media URLs inaccessible or expired
- **Content Policy Violations**: Messages/media flagged by Discord moderation
- **Emoji/Sticker Conflicts**: Name collisions with existing assets

### Message and Threading Issues
- **Message Not Found**: References to deleted or inaccessible messages
- **Thread Creation Failures**: Attempting to create threads on inappropriate messages
- **Channel Type Mismatch**: Operations not supported in specific channel types
- **Server Boost Requirements**: Features requiring specific server boost levels

## Technical Debt

### Error Handling Complexity
- Generic error messages for diverse Discord API failure modes
- Limited retry logic for transient API failures
- Poor error context for permission-related failures
- Insufficient validation of Discord-specific constraints

### Configuration Management
- Complex action gating system requiring manual configuration
- No automated permission validation during setup
- Limited guidance for optimal permission configuration
- Missing configuration migration tools between Discord API versions

### Media Processing Limitations
- No intelligent resizing or format conversion for uploads
- Basic file validation without content analysis
- No preview generation for media attachments
- Limited support for animated content optimization

### Integration Complexity
- Heavy reliance on Discord API stability and behavior
- No graceful degradation for deprecated API features
- Complex permission inheritance model from Discord
- Limited abstraction over Discord's evolving feature set

## Dependencies

### Core Dependencies
- **Discord.js Library**: Node.js Discord API client library
- **OpenClaw Message System**: Integration with generic messaging framework
- **HTTP Client**: For file uploads and external URL handling

### Authentication Dependencies
- **Bot Token**: Discord application bot token for API access
- **Gateway Configuration**: `channels.discord` configuration section
- **Webhook Support**: For real-time event handling (optional)

### Media Processing Dependencies
- **File System**: Access to local file storage for uploads
- **HTTP Client**: Remote URL file fetching
- **Image Processing**: Basic validation of media file formats
- **Streaming**: Large file upload handling

### Discord API Dependencies
- **Discord Gateway**: WebSocket connection for real-time events
- **REST API**: HTTP API for standard Discord operations
- **CDN Access**: Content delivery network for media assets
- **OAuth2**: Authentication flow for advanced integrations

### Optional Dependencies
- **Database**: Persistent storage for custom commands and data
- **Cache**: Redis or similar for rate limit management
- **Notification System**: External alerting for moderation events
- **Analytics**: Usage tracking and performance monitoring

### Development Dependencies
- **TypeScript**: Type safety for Discord API interactions
- **Testing Framework**: Mock Discord API for automated testing
- **Linting**: Code quality enforcement for Discord integration
- **Documentation**: API documentation generation tools