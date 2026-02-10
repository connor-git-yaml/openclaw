# Extension Specification: WhatsApp

## Intent

WhatsApp extension enables OpenClaw integration with WhatsApp messaging platform through WhatsApp Business API or unofficial web interfaces. It provides messaging capabilities for the world's most popular messaging platform, supporting text messages, media sharing, group communications, and business messaging features where applicable.

## Interface

### Plugin Registration
- **ID**: `whatsapp`
- **Name**: WhatsApp
- **Type**: Channel Plugin
- **API Surface**: Channel plugin with WhatsApp integration

### Channel Plugin Methods
- `sendMessage()` - Send text messages to contacts or groups
- `sendMedia()` - Share images, videos, documents, and voice messages
- `getContacts()` - Retrieve WhatsApp contact list
- `joinGroup()` - Join WhatsApp groups via invite links
- `handleStatus()` - Process read receipts and typing indicators

### WhatsApp Features
- **Business API**: Official WhatsApp Business API integration
- **Web Interface**: WhatsApp Web automation interface
- **Media Support**: Comprehensive media type support
- **Group Management**: Group creation, administration, and participation
- **Contact Management**: Access to WhatsApp contact directory

## Business Logic

### Integration Methods
- **Business API**: Official API for business use cases
- **Web Automation**: Browser automation of WhatsApp Web
- **Desktop Client**: Integration with WhatsApp Desktop application
- **Third-party APIs**: Unofficial API services and libraries

### Message Processing
- **Text Formatting**: Markdown-style text formatting support
- **Media Handling**: Images, videos, audio, documents, and stickers
- **Contact Cards**: Share contact information and vCards
- **Location Sharing**: Send and receive location information

### Authentication & Setup
- **Phone Number**: Link to verified phone number
- **QR Code**: QR code scanning for web-based authentication
- **Business Account**: Business account verification and setup
- **API Credentials**: Manage API keys and webhook configurations

## Data Structures

### Configuration
```typescript
WhatsAppConfig {
  phoneNumber: string;
  apiKey?: string; // For Business API
  webhookUrl?: string;
  businessAccountId?: string;
  webInterface?: boolean; // Use web automation
}
```

### Message Format
```typescript
WhatsAppMessage {
  id: string;
  chatId: string; // Phone number or group ID
  sender: string;
  type: "text" | "image" | "video" | "audio" | "document";
  content: string | MediaContent;
  timestamp: number;
  status: "sent" | "delivered" | "read";
}
```

## Constraints

### Platform Limitations
- **Phone Number Dependency**: Requires verified phone number
- **Single Device**: Limited multi-device support
- **Business API Costs**: Official API has usage costs
- **Rate Limiting**: Message sending rate limits

### Technical Constraints
- **Unofficial APIs**: Risk of API changes or blocking
- **Web Automation**: Browser dependency for web-based integration
- **Media Limits**: File size and type restrictions
- **Group Limits**: Maximum group size and admin restrictions

## Edge Cases

### Authentication Issues
- **Phone Verification**: Failed phone number verification
- **QR Code Expiry**: Expired QR codes requiring re-scan
- **Account Ban**: WhatsApp account suspension or blocking
- **API Changes**: Breaking changes in unofficial APIs

### Message Delivery
- **Offline Recipients**: Messages to offline contacts
- **Blocked Contacts**: Messages to contacts who blocked the account
- **Network Issues**: Connectivity problems affecting delivery
- **Media Upload Failures**: Large file upload timeouts

## Technical Debt

### Integration Stability
- **Unofficial API Risk**: Dependence on unofficial APIs subject to change
- **Web Automation Brittleness**: Browser automation fragility
- **Error Handling**: Complex error scenarios from various integration methods
- **Update Management**: WhatsApp client updates affecting integration

### Performance Issues
- **Browser Overhead**: Performance impact of browser automation
- **Media Processing**: Large media file handling optimization
- **Contact Sync**: Efficient contact list synchronization
- **Message History**: Handling large conversation histories

## Dependencies

### External Dependencies
- **WhatsApp Service**: WhatsApp messaging infrastructure
- **WhatsApp Business API**: Official business messaging API
- **Browser/WebDriver**: For web automation approach
- **WhatsApp Desktop**: For desktop client integration

### Internal Dependencies
- **openclaw/plugin-sdk**: Core channel framework
- **HTTP Client**: API communication
- **Media Processing**: File and media handling
- **Browser Automation**: Web scraping and automation tools