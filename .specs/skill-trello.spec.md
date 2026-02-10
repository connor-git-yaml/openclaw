# Trello Skill Specification

## Intent

Manage Trello boards, lists, and cards through the Trello REST API for project management, task tracking, and collaborative workflow automation.

## Interface

### Core Operations
- **Board Management**: Create, update, and organize Trello boards
- **List Operations**: Manage board lists and list positioning
- **Card Management**: Create, update, move, and archive cards
- **Member Management**: Add/remove members and handle permissions
- **Label and Due Date**: Apply labels, due dates, and card metadata

### API Integration
- **REST API**: Direct Trello API calls using curl and jq
- **Authentication**: API key and token-based authentication
- **Data Processing**: JSON parsing and manipulation with jq

## Business Logic

### Board and List Organization
1. Create and configure boards with appropriate settings
2. Manage list structures and workflow stages
3. Handle board permissions and member access
4. Process board templates and organizational patterns

### Card Lifecycle Management
1. Create cards with descriptions, due dates, and labels
2. Move cards between lists representing workflow stages  
3. Update card metadata and progress tracking
4. Archive completed cards and manage card history

### Collaboration and Member Management
1. Add team members to boards and cards
2. Assign cards to specific members
3. Handle notification settings and activity tracking
4. Manage board visibility and access permissions

## Data Structures

### Trello Board
```typescript
interface TrelloBoard {
  id: string; // board identifier
  name: string; // board name
  desc?: string; // board description
  closed: boolean; // archived status
  url: string; // board URL
  lists: TrelloList[]; // board lists
  members: Member[]; // board members
  labels: Label[]; // board labels
}
```

### Trello List and Card
```typescript
interface TrelloList {
  id: string; name: string; pos: number;
  closed: boolean; board: string;
}

interface TrelloCard {
  id: string; name: string; desc?: string;
  due?: Date; closed: boolean;
  list: string; board: string;
  members: string[]; labels: string[];
  url: string; pos: number;
}
```

### Authentication
```typescript
interface TrelloAuth {
  apiKey: string; // TRELLO_API_KEY
  token: string; // TRELLO_TOKEN
  baseUrl: 'https://api.trello.com/1';
}
```

## Constraints

### API Limitations
- **Rate Limiting**: 300 requests per 10 seconds per token
- **Data Limits**: Maximum card description and comment lengths
- **Member Limits**: Board member quantity restrictions
- **Attachment Limits**: File size and quantity restrictions

### Authentication Requirements
- **API Key**: Trello application API key required
- **User Token**: User authorization token for account access
- **Permissions**: Token scope determines available operations

## Edge Cases

### Authentication Issues
- **Invalid Credentials**: Expired or incorrect API keys/tokens
- **Permission Denied**: Insufficient token permissions for operations
- **Account Restrictions**: Trello account limitations or suspension

### Data Synchronization
- **Concurrent Updates**: Multiple users modifying same resources
- **Stale Data**: Cached information becoming outdated
- **Network Issues**: API connectivity problems affecting operations

## Technical Debt

### Manual API Integration
- Direct curl commands without sophisticated error handling
- Limited retry logic for transient API failures
- Basic JSON processing without advanced validation

### Configuration Management
- Manual credential management without secure storage
- No automatic token refresh or validation
- Limited error reporting for configuration issues

## Dependencies

### Core Dependencies
- **jq**: JSON processing for API responses
- **curl**: HTTP client for API requests
- **TRELLO_API_KEY**: Trello application API key
- **TRELLO_TOKEN**: User authorization token

### System Dependencies
- **Network Connectivity**: Internet access for Trello API
- **Shell Environment**: Command execution capabilities
- **JSON Processing**: Response parsing and manipulation