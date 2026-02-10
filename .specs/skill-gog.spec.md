# Gog Skill Specification

## Intent

Provide comprehensive command-line interface to Google Workspace services including Gmail, Calendar, Drive, Contacts, Sheets, and Docs. Enables email management, calendar scheduling, document collaboration, and data processing through unified CLI with OAuth authentication.

## Interface

### Service Coverage
- **Gmail**: Email search, send, drafts, replies, threading management
- **Calendar**: Event creation, management, scheduling, color coordination
- **Drive**: File search, access, sharing, metadata management
- **Contacts**: Contact list access and management
- **Sheets**: Spreadsheet reading, writing, formatting, data manipulation
- **Docs**: Document export, content access, basic operations

### Command Structure
- `gog gmail send --to EMAIL --subject TEXT --body TEXT` - Send email
- `gog calendar create CAL_ID --summary TEXT --from ISO --to ISO` - Create event
- `gog drive search "QUERY" --max N` - Search files
- `gog contacts list --max N` - List contacts
- `gog sheets get SHEET_ID "RANGE" --json` - Read spreadsheet data
- `gog docs export DOC_ID --format txt --out FILE` - Export document

### Authentication Management
- `gog auth credentials /path/to/client_secret.json` - Configure OAuth
- `gog auth add EMAIL --services gmail,calendar,drive` - Add account
- `gog auth list` - Show authenticated accounts

### Output Formats
- Human-readable default formatting
- JSON output (`--json`) for programmatic access
- File output (`--out FILE`) for data export

## Business Logic

### OAuth Authentication Flow
1. Configure Google Cloud OAuth credentials from client secret file
2. Add user accounts with specific service scope permissions
3. Handle OAuth token refresh and expiration automatically
4. Manage multiple account authentication and switching

### Email Management Workflow
1. Compose emails with plain text, HTML, or file input
2. Search emails with Gmail query syntax and threading
3. Create and manage drafts for later sending
4. Handle replies with proper message threading
5. Support attachments and rich formatting options

### Calendar Coordination System
1. Create events with scheduling conflict detection
2. Apply color coding for visual organization and categorization
3. Manage recurring events and meeting series
4. Coordinate across multiple calendars and accounts
5. Export calendar data for external processing

### Document and Data Processing
1. Search and access Google Drive files with query filters
2. Read and write Google Sheets with range specifications
3. Export Google Docs in multiple formats (txt, html, pdf)
4. Handle batch operations for data processing workflows
5. Manage sharing permissions and collaboration settings

## Data Structures

### Google Account
```typescript
interface GoogleAccount {
  email: string;               // account email address
  services: GoogleService[];   // authorized service scopes
  tokenExpiry: Date;           // OAuth token expiration
  lastUsed: Date;              // last activity timestamp
  isDefault: boolean;          // default account for operations
}

type GoogleService = 'gmail' | 'calendar' | 'drive' | 'contacts' | 'docs' | 'sheets';
```

### Email Message
```typescript
interface EmailMessage {
  id: string;                  // message identifier
  threadId: string;            // conversation thread
  subject: string;             // email subject line
  from: string;                // sender address
  to: string[];                // recipient addresses
  cc?: string[];               // carbon copy recipients
  body: string;                // message content
  isHtml: boolean;             // HTML vs plain text
  attachments: Attachment[];   // file attachments
  timestamp: Date;             // send/receive time
  labels: string[];            // Gmail labels
}
```

### Calendar Event
```typescript
interface CalendarEvent {
  id: string;                  // event identifier
  calendarId: string;          // containing calendar
  summary: string;             // event title
  description?: string;        // event details
  startTime: Date;             // event start
  endTime: Date;               // event end
  location?: string;           // event location
  attendees: Attendee[];       // invited participants
  colorId?: number;            // color coding (1-11)
  recurrence?: string[];       // recurring event rules
  reminders: Reminder[];       // notification settings
}
```

### Sheets Data
```typescript
interface SheetsData {
  sheetId: string;             // spreadsheet identifier
  range: string;               // cell range (A1:B10 format)
  values: any[][];             // cell data matrix
  majorDimension: 'ROWS' | 'COLUMNS';
  valueInputOption: 'RAW' | 'USER_ENTERED';
  metadata?: SheetMetadata;    // spreadsheet structure info
}
```

### Drive File
```typescript
interface DriveFile {
  id: string;                  // file identifier
  name: string;                // file name
  mimeType: string;            // file type
  size?: number;               // file size in bytes
  createdTime: Date;           // creation timestamp
  modifiedTime: Date;          // last modification
  owners: string[];            // file owners
  permissions: Permission[];   // sharing settings
  parentFolders: string[];     // containing folders
  webViewLink?: string;        // browser view URL
}
```

## Constraints

### Google API Limitations
- **Rate Limits**: Each service has specific request quotas and frequency limits
- **Scope Permissions**: Operations limited by OAuth scope grants
- **Data Limits**: Gmail has sending limits, Sheets has cell limits, etc.
- **API Versions**: Google services may deprecate older API versions

### Authentication Requirements
- **OAuth Setup**: Requires Google Cloud Console project with OAuth credentials
- **Service Enablement**: Each Google service must be enabled in Cloud Console
- **Token Management**: OAuth tokens expire and require refresh handling
- **Account Limits**: Google imposes limits on concurrent OAuth sessions

### Data Format Constraints
- **Gmail Query Syntax**: Search queries must follow Gmail's specific format
- **Sheets Range Format**: Cell ranges require A1 notation (e.g., "Sheet1!A1:B10")
- **Calendar DateTime**: Events require ISO 8601 datetime format
- **File Size Limits**: Drive operations subject to file size restrictions

### Service Dependencies
- **Internet Connectivity**: All operations require stable internet access
- **Google Service Status**: Operations fail during Google service outages
- **Account Status**: Google account suspension affects all service access
- **Billing Limits**: Exceeding API quotas may suspend access

## Edge Cases

### Authentication and Authorization
- **Token Expiry**: OAuth tokens expiring during long operations
- **Scope Changes**: Google modifying required permissions for operations
- **Account Suspension**: Google account locked or restricted
- **Multi-Factor Authentication**: 2FA interfering with CLI authentication
- **Corporate Restrictions**: Organizational policies blocking OAuth flows

### API and Service Issues
- **Rate Limit Exceeded**: Too many requests causing temporary blocks
- **Service Degradation**: Google services running slowly or partially unavailable
- **API Changes**: Google modifying API behavior or deprecating features
- **Quota Exhausted**: Daily/monthly usage limits reached
- **Regional Restrictions**: Services unavailable in specific geographic regions

### Data and Content Problems
- **Large Datasets**: Sheets operations failing with massive data volumes
- **Email Size Limits**: Gmail rejecting large emails or attachments
- **Format Incompatibility**: Docs/Sheets export failing for complex formats
- **Permission Denied**: Insufficient access rights for specific operations
- **Content Policy Violations**: Google blocking content that violates policies

### Network and Connectivity
- **Internet Outage**: Network connectivity loss during operations
- **Proxy Issues**: Corporate firewalls blocking Google API access
- **DNS Problems**: Unable to resolve Google service domains
- **SSL/TLS Errors**: Certificate validation failures
- **Timeout Issues**: Slow networks causing request timeouts

## Technical Debt

### Error Handling and Recovery
- Generic error messages without specific guidance for Google API errors
- No automatic retry logic for transient failures
- Limited offline capabilities when Google services unavailable
- Poor error context for permission and authentication failures

### Configuration Management
- OAuth credential management basic and potentially insecure
- No configuration validation or setup wizard for Google Cloud setup
- Limited account switching and multi-account coordination
- Missing configuration backup and migration tools

### Performance and Optimization
- No local caching of frequently accessed data (contacts, calendar events)
- Sequential processing instead of batch operations where possible
- Inefficient handling of large dataset operations
- No progressive loading for large data responses

### Feature Coverage Gaps
- Limited Google Docs editing capabilities (read-only mostly)
- No support for advanced Google Sheets functions and formatting
- Missing Google Drive file management operations (upload, delete)
- No integration with newer Google Workspace services (Meet, Chat)

## Dependencies

### Core Dependencies
- **gog Binary**: Primary CLI tool for Google Workspace integration
- **HTTP Client**: For Google API communication
- **JSON Processing**: Request/response serialization and parsing
- **OAuth2 Client**: Authentication flow implementation

### Google API Dependencies
- **Gmail API**: Email operations and management
- **Calendar API**: Event scheduling and calendar access
- **Drive API**: File storage and sharing operations
- **Contacts API**: Contact management and access
- **Sheets API**: Spreadsheet data manipulation
- **Docs API**: Document access and export

### Authentication Dependencies
- **Google Cloud Console**: OAuth credential configuration
- **OAuth2 Flow**: User authentication and authorization
- **Token Storage**: Secure credential persistence
- **Service Account**: Alternative authentication for server environments

### System Dependencies
- **Configuration Storage**: File system access for credentials and settings
- **Network Stack**: HTTPS connectivity for all Google API operations
- **Process Management**: Command execution and output handling
- **File I/O**: Document export and data file operations

### Optional Dependencies
- **Homebrew**: Package manager for macOS installation
- **Text Editors**: For composing email body content
- **File Managers**: For handling document exports and attachments
- **JSON Processors**: jq for advanced JSON data manipulation

### Development Dependencies
- **Go Toolchain**: For building gog from source
- **Testing Framework**: Automated testing of Google API integrations
- **Documentation Tools**: Help system and command reference generation
- **Build System**: Cross-platform compilation and distribution