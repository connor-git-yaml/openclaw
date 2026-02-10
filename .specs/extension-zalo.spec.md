# Extension Specification: Zalo

## Intent

Zalo extension provides integration with Zalo messaging platform, Vietnam's leading social messaging service. It enables OpenClaw to send and receive messages, manage contacts, participate in group chats, and utilize Zalo's rich messaging features through official or unofficial API integrations, serving the Vietnamese market and Southeast Asian users.

## Interface

### Plugin Registration
- **ID**: `zalo`
- **Name**: Zalo
- **Type**: Channel Plugin
- **API Surface**: Channel plugin with Zalo messaging integration

### Channel Plugin Methods
- `sendMessage()` - Send text messages to contacts or groups
- `sendMedia()` - Share images, videos, and file attachments
- `manageContacts()` - Access and manage Zalo contact list
- `joinGroup()` - Participate in Zalo group conversations
- `handleStickers()` - Send and receive Zalo stickers and emoticons

### Zalo Features
- **Rich Messaging**: Text, media, stickers, and location sharing
- **Group Management**: Create and manage Zalo group chats
- **Official Accounts**: Integration with Zalo Official Accounts (ZOA)
- **Mini Apps**: Integration with Zalo Mini App ecosystem
- **Payment Integration**: Zalo Pay payment and transaction features

## Business Logic

### Authentication & Setup
- **Phone Number**: Link to verified Vietnamese phone number
- **API Integration**: Official Zalo API or third-party interfaces
- **OA Management**: Official Account setup and management
- **App Permissions**: Zalo application permissions and scopes

### Message Processing
- **Text Formatting**: Vietnamese text support and formatting
- **Media Handling**: Images, videos, audio, and document sharing
- **Sticker System**: Zalo's extensive sticker and emoji collection
- **Location Services**: Location sharing and map integration

### Group Operations
- **Group Creation**: Create and configure group conversations
- **Member Management**: Add, remove, and manage group members
- **Admin Functions**: Group administration and moderation tools
- **Group Settings**: Configure privacy and notification settings

## Data Structures

### Configuration
```typescript
ZaloConfig {
  phoneNumber: string;
  apiKey?: string; // For official API
  appId?: string; // Zalo app identification
  oaId?: string; // Official Account ID
  accessToken?: string;
}
```

### Message Format
```typescript
ZaloMessage {
  msgId: string;
  fromId: string; // Sender ID
  toId: string; // Recipient ID (user or group)
  msgType: "text" | "image" | "video" | "sticker" | "file";
  content: string | MediaContent;
  timestamp: number;
  groupId?: string;
}
```

### Contact Information
```typescript
ZaloContact {
  userId: string;
  displayName: string;
  phoneNumber?: string;
  avatar?: string;
  status: "online" | "offline" | "away";
  isFriend: boolean;
}
```

## Constraints

### Regional Limitations
- **Geographic Focus**: Primarily serves Vietnamese and Southeast Asian markets
- **Language Support**: Optimized for Vietnamese language and culture
- **Phone Verification**: Requires Vietnamese phone number for full functionality
- **Regulatory Compliance**: Subject to Vietnamese internet regulations

### Technical Constraints
- **API Availability**: Limited official API access for third-party developers
- **Rate Limiting**: Message sending rate limits and quotas
- **Media Restrictions**: File size and format limitations
- **Group Limits**: Maximum group size and member restrictions

### Platform Dependencies
- **Mobile-First**: Designed primarily for mobile device usage
- **App Installation**: May require Zalo mobile app for certain features
- **Network Requirements**: Optimized for Southeast Asian network conditions
- **Account Verification**: Complex verification process for business accounts

## Edge Cases

### Authentication Issues
- **Phone Verification**: Failed Vietnamese phone number verification
- **Account Suspension**: Zalo account suspension or restrictions
- **API Access**: Limited or revoked API access permissions
- **Regional Blocking**: Geographic restrictions on API access

### Message Delivery
- **Offline Recipients**: Messages to users not currently online
- **Blocked Contacts**: Messages to users who have blocked the sender
- **Group Restrictions**: Message delivery issues in large groups
- **Media Upload Failures**: Failed media uploads due to size or format

### Cultural and Language
- **Text Encoding**: Vietnamese text encoding and display issues
- **Cultural Context**: Cultural appropriateness of automated messages
- **Local Holidays**: Message delivery during Vietnamese holidays
- **Time Zone**: Vietnam time zone handling and scheduling

## Technical Debt

### API Integration Challenges
- **Unofficial API Risk**: Dependence on reverse-engineered or unofficial APIs
- **Documentation**: Limited English documentation for Zalo APIs
- **Error Handling**: Vietnamese error messages and localized responses
- **Update Frequency**: Frequent Zalo app updates affecting integration

### Localization Needs
- **Language Support**: Comprehensive Vietnamese language processing
- **Cultural Adaptation**: Vietnamese social norms and messaging etiquette
- **Local Services**: Integration with Vietnam-specific services and payment
- **Regional Testing**: Testing with Vietnamese users and network conditions

### Performance Optimization
- **Network Optimization**: Optimization for Southeast Asian network infrastructure
- **Image Compression**: Efficient image compression for slower connections
- **Caching Strategy**: Local caching for frequently used stickers and media
- **Offline Handling**: Robust offline message queuing and delivery

## Dependencies

### External Dependencies
- **Zalo Platform**: Zalo messaging service infrastructure
- **Vietnamese Telecom**: Vietnamese telecom providers for SMS verification
- **Zalo APIs**: Official or unofficial Zalo API services
- **Payment Systems**: Zalo Pay and Vietnamese payment integration

### Internal Dependencies
- **openclaw/plugin-sdk**: Core channel framework
- **HTTP Client**: API communication with Zalo services
- **Media Processing**: Vietnamese text and media file handling
- **Localization**: Vietnamese language and cultural localization