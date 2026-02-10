# Extension Specification: LINE

## Intent

LINE extension provides integration with LINE Messaging API, enabling OpenClaw to communicate through LINE's popular messaging platform. It implements LINE's Bot Framework for sending and receiving messages, handling rich cards, managing webhooks, and supporting LINE's unique features like stickers, quick replies, and template messages. The extension facilitates AI assistant deployment within LINE's ecosystem, particularly popular in Asia-Pacific markets.

## Interface

### Plugin Registration
- **ID**: `line`
- **Name**: LINE
- **Type**: Channel Plugin
- **Entry Point**: `index.ts` exports plugin definition with card command registration
- **API Surface**: Channel plugin, webhook handler, card command system

### Channel Plugin Methods
- `sendMessage()` - Send text and rich messages to users or groups
- `sendCard()` - Send template messages and carousel cards
- `uploadMedia()` - Upload and share images, videos, audio files
- `sendSticker()` - Send LINE stickers and emoji
- `sendQuickReply()` - Interactive quick reply buttons

### LINE-Specific Features
- **Template Messages**: Buttons, carousels, confirm dialogs
- **Rich Menus**: Persistent bottom menu with custom actions
- **LINE Stickers**: Native sticker support with official sticker packages
- **Quick Replies**: Interactive response options
- **Flex Messages**: Custom layout messages with advanced formatting

### Card Command Integration
- **Custom Commands**: LINE-specific card templates and interactions
- **Action Handling**: Process user taps on buttons and quick replies
- **Template Generation**: Dynamic card creation for LINE's format requirements

## Business Logic

### LINE Bot Framework
- **Channel Access Token**: Authentication with LINE Messaging API
- **Webhook Events**: Real-time event processing from LINE platform
- **Message Types**: Text, image, video, audio, location, sticker support
- **User Management**: Friend/follower relationship handling

### Rich Message Processing
- **Template Messages**: Button templates, carousel templates, confirm templates
- **Flex Messages**: Custom JSON-based layout system for complex UIs
- **Quick Reply Integration**: Interactive response options with payloads
- **Action Processing**: Handle postback actions, URI actions, message actions

### Media and Content Handling
- **Media Upload**: Images, videos, and audio files through LINE's CDN
- **Content Delivery**: Temporary media URLs with expiration handling
- **Sticker Integration**: Official and custom sticker package support
- **File Size Management**: LINE's file size and duration limitations

### Event Management
- **Message Events**: Text, image, video, audio, location, sticker events
- **Follow Events**: User follows/unfollows bot
- **Postback Events**: User interactions with template message buttons
- **Beacon Events**: LINE Beacon proximity-based interactions

## Data Structures

### LINE Configuration
```typescript
LineAccountConfig {
  name?: string;
  channelAccessToken?: string;
  channelSecret?: string;
  webhookPath?: string;
  richMenuId?: string;
  capabilities?: string[];
  dmPolicy?: DmPolicy;
  groupPolicy?: GroupPolicy;
}
```

### Message Types
```typescript
LineMessage {
  id: string;
  type: "text" | "image" | "video" | "audio" | "file" | "location" | "sticker";
  text?: string;
  contentProvider?: ContentProvider;
  duration?: number;
  packageId?: string; // For stickers
  stickerId?: string; // For stickers
}
```

### Template Messages
```typescript
ButtonTemplate {
  type: "buttons";
  thumbnailImageUrl?: string;
  imageAspectRatio?: "rectangle" | "square";
  imageSize?: "cover" | "contain";
  imageBackgroundColor?: string;
  title?: string;
  text: string;
  defaultAction?: Action;
  actions: Action[];
}

CarouselTemplate {
  type: "carousel";
  columns: CarouselColumn[];
  imageAspectRatio?: "rectangle" | "square";
  imageSize?: "cover" | "contain";
}
```

### Event Structures
```typescript
WebhookEvent {
  type: "message" | "follow" | "unfollow" | "postback" | "beacon";
  source: EventSource;
  timestamp: number;
  message?: LineMessage;
  postback?: PostbackData;
  beacon?: BeaconData;
}
```

## Constraints

### LINE API Limitations
- **Rate Limits**: Strict message sending quotas (varies by plan)
- **Message Size**: Text content limited to 5,000 characters
- **Media Limits**: File size restrictions for images, videos, audio
- **Template Complexity**: Limits on carousel columns and button counts

### Authentication Requirements
- **Channel Access Token**: Required for API authentication
- **Channel Secret**: Required for webhook signature verification
- **LINE Developer Account**: Must be registered in LINE Developers console
- **App Verification**: Business verification may be required for certain features

### Feature Restrictions
- **Group Messaging**: Limited functionality in group chats
- **Broadcasting**: Restrictions on mass messaging to users
- **Rich Menu**: Limited to specific message types and user contexts
- **Sticker Packages**: Only official stickers available without special approval

### Regional Considerations
- **Geographic Availability**: LINE service availability varies by region
- **Local Compliance**: Data residency and privacy regulations
- **Language Support**: Right-to-left and complex script support limitations
- **Cultural Features**: Region-specific features and sticker packages

## Edge Cases

### Authentication Issues
- **Token Expiration**: Channel access tokens requiring renewal
- **Webhook Verification**: Signature validation failures
- **Permission Denied**: Insufficient bot permissions for requested actions
- **Account Suspension**: LINE account or bot suspension scenarios

### Message Delivery
- **User Blocks Bot**: Users blocking the bot affecting message delivery
- **Rate Limit Exceeded**: Temporary sending restrictions due to quota exhaustion
- **Invalid Recipients**: Messages to non-existent or deactivated accounts
- **Group Restrictions**: Group administrators restricting bot functionality

### Rich Content Issues
- **Template Validation**: Complex template messages failing validation
- **Media Upload Failures**: CDN upload errors for images and videos
- **Flex Message Errors**: Invalid JSON structure in custom layouts
- **Action Payload Limits**: Postback data exceeding size limitations

### Network and Infrastructure
- **Webhook Delivery**: Failed webhook delivery requiring retry mechanisms
- **API Downtime**: LINE Messaging API service outages
- **Geographic Routing**: API endpoint routing issues in different regions
- **CDN Issues**: Media content delivery network problems

## Technical Debt

### Code Organization
- **Template Complexity**: Complex nested structures for rich messages
- **Event Handling**: Mixed event processing patterns across different event types
- **Media Management**: Scattered media upload and processing logic
- **Error Handling**: Inconsistent error handling for different LINE API responses

### Performance Issues
- **Synchronous Processing**: Blocking webhook processing for complex templates
- **Memory Usage**: Large template messages and media content consuming memory
- **API Efficiency**: Suboptimal API call patterns for bulk operations
- **Cache Management**: Missing caching for frequently used stickers and templates

### Testing Challenges
- **Webhook Testing**: Difficult to test real-time webhook scenarios
- **Template Validation**: Complex validation logic for various template types
- **Rich Menu Testing**: Limited testing options for rich menu interactions
- **Sticker Testing**: Testing sticker functionality across different packages

### Documentation and Maintenance
- **LINE API Changes**: Frequent LINE API updates requiring code maintenance
- **Template Documentation**: Complex template format documentation and examples
- **Regional Differences**: Documentation for region-specific features and limitations
- **Debugging Tools**: Limited debugging capabilities for template and webhook issues

## Dependencies

### External Dependencies
- **LINE Messaging API**: Core messaging service and webhook delivery
- **LINE Developers Platform**: Bot registration and management console
- **LINE CDN**: Content delivery network for media files
- **LINE Official Account**: Business account features and verification

### Internal Dependencies
- **openclaw/plugin-sdk**: Core channel framework and message processing
- **HTTP Server**: Webhook endpoint hosting and signature verification
- **Media Processing**: Image and video processing for uploads
- **Template Engine**: Card and template message generation system

### Runtime Requirements
- **Network Connectivity**: Stable connection to LINE Messaging API endpoints
- **HTTPS Support**: SSL/TLS for webhook endpoints (required by LINE)
- **Storage**: Media file storage and template caching
- **Memory**: Sufficient memory for processing complex template messages and media files