# Himalaya Skill Specification

## Intent

Provide comprehensive terminal-based email management through the Himalaya CLI client. Supports IMAP/SMTP email operations including reading, writing, searching, organizing, and multi-account management with secure authentication and advanced composition capabilities.

## Interface

### Core Email Operations
- **Message Management**: Read, write, reply, forward, delete messages
- **Organization**: Move, copy, flag, and folder management
- **Search and Discovery**: Advanced email search with filtering
- **Composition**: Rich email composition with MML (MIME Meta Language) support
- **Attachments**: Download, save, and manage email attachments

### Command Structure
- `himalaya envelope list [--folder FOLDER]` - List emails with pagination
- `himalaya message read ID` - Display email content
- `himalaya message write [-H HEADER]` - Compose new email
- `himalaya message reply ID [--all]` - Reply to message
- `himalaya message move ID FOLDER` - Organize emails
- `himalaya attachment download ID [--dir PATH]` - Handle attachments

### Account Management
- `himalaya account configure` - Interactive account setup
- `himalaya account list` - Show configured accounts
- `himalaya --account NAME COMMAND` - Account-specific operations

### Output Formats
- Human-readable default formatting
- JSON output (`--output json`) for programmatic access
- Plain text output (`--output plain`) for scripting

## Business Logic

### Authentication and Security Framework
1. Support multiple authentication methods (password, OAuth, command-based)
2. Secure credential storage via system keyring or password managers
3. Handle session management and token refresh for IMAP/SMTP connections
4. Encrypt sensitive configuration data and communication

### Email Processing Pipeline
1. Retrieve email envelopes via IMAP with efficient pagination
2. Parse MIME messages with proper character encoding handling
3. Process attachments with format detection and extraction
4. Handle message threading and conversation grouping
5. Support rich formatting and HTML content processing

### Multi-Account Coordination
1. Manage multiple email accounts with independent configurations
2. Support account switching and cross-account operations
3. Handle different authentication requirements per account
4. Coordinate folder structures and synchronization across accounts

### Advanced Composition System
1. MML (MIME Meta Language) support for rich email composition
2. External editor integration for message writing
3. Template-based message generation with variable substitution
4. Attachment embedding and MIME part handling
5. Message validation and delivery confirmation

## Data Structures

### Email Account
```typescript
interface EmailAccount {
  name: string;                // account identifier
  email: string;               // email address
  displayName: string;         // sender display name
  isDefault: boolean;          // default account flag
  backend: IMAPConfig;         // IMAP configuration
  sendBackend: SMTPConfig;     // SMTP configuration
  folders: EmailFolder[];      // available folders
  lastSync: Date;              // last synchronization
}
```

### IMAP/SMTP Configuration
```typescript
interface IMAPConfig {
  type: 'imap';
  host: string;                // IMAP server hostname
  port: number;                // connection port
  encryption: 'tls' | 'start-tls' | 'none';
  login: string;               // authentication username
  auth: AuthMethod;            // authentication configuration
}

interface SMTPConfig {
  type: 'smtp';
  host: string;                // SMTP server hostname
  port: number;                // connection port
  encryption: 'tls' | 'start-tls' | 'none';
  login: string;               // authentication username
  auth: AuthMethod;            // authentication configuration
}

interface AuthMethod {
  type: 'password' | 'oauth2' | 'xoauth2';
  cmd?: string;                // command to retrieve password
  keyring?: boolean;           // use system keyring
}
```

### Email Message
```typescript
interface EmailMessage {
  id: number;                  // folder-relative message ID
  uid: string;                 // IMAP UID for message
  subject: string;             // email subject
  from: EmailAddress[];        // sender information
  to: EmailAddress[];          // primary recipients
  cc?: EmailAddress[];         // carbon copy recipients
  bcc?: EmailAddress[];        // blind carbon copy recipients
  date: Date;                  // message timestamp
  flags: MessageFlag[];        // IMAP flags
  size: number;                // message size in bytes
  attachments: Attachment[];   // file attachments
  bodyText?: string;           // plain text content
  bodyHtml?: string;           // HTML content
}
```

### Email Folder
```typescript
interface EmailFolder {
  name: string;                // folder name
  path: string;                // folder path
  messageCount: number;        // total messages
  unreadCount: number;         // unread message count
  attributes: FolderAttribute[]; // IMAP folder attributes
  delimiter: string;           // hierarchy delimiter
}

type FolderAttribute = 'HasNoChildren' | 'HasChildren' | 'Inbox' | 'Sent' | 'Drafts' | 'Trash';
type MessageFlag = 'Seen' | 'Answered' | 'Flagged' | 'Deleted' | 'Draft' | 'Recent';
```

### Search Query
```typescript
interface SearchQuery {
  from?: string;               // sender filter
  to?: string;                 // recipient filter
  subject?: string;            // subject filter
  body?: string;               // body text search
  since?: Date;                // date range start
  before?: Date;               // date range end
  flags?: MessageFlag[];       // flag filters
  folder?: string;             // folder restriction
}
```

## Constraints

### Protocol and Server Limitations
- **IMAP Compatibility**: Requires IMAP-compatible email servers
- **SMTP Authentication**: SMTP servers must support authentication methods
- **Connection Limits**: Email providers may limit concurrent connections
- **Message Size**: Large emails may exceed server or client limits

### Security and Authentication Requirements
- **Credential Storage**: Secure storage for email passwords and tokens
- **SSL/TLS Support**: Encrypted connections required for most providers
- **OAuth Complexity**: Modern authentication flows require additional setup
- **Two-Factor Authentication**: May require app-specific passwords

### Terminal and Interface Constraints
- **Terminal Compatibility**: Rich display features require capable terminal
- **External Editor**: Message composition requires configured editor
- **Character Encoding**: Proper Unicode support needed for international content
- **Color Support**: Optimal experience requires color terminal support

### File System and Storage
- **Configuration Directory**: Write access to `~/.config/himalaya/`
- **Temporary Files**: Space for message composition and attachment handling
- **Cache Storage**: Local storage for offline message access
- **Attachment Downloads**: Configurable download directory access

## Edge Cases

### Network and Connectivity Issues
- **Server Unavailable**: IMAP/SMTP servers offline or unreachable
- **Authentication Failures**: Invalid credentials or expired tokens
- **Connection Timeouts**: Slow networks causing operation failures
- **Partial Synchronization**: Interrupted sync leaving incomplete state
- **Certificate Problems**: SSL/TLS certificate validation failures

### Message and Content Problems
- **Corrupted Messages**: MIME parsing failures for malformed emails
- **Large Attachments**: Memory issues with oversized file attachments
- **Character Encoding**: Display problems with non-UTF8 content
- **HTML Complexity**: Rich HTML emails not rendering properly in terminal
- **Thread Confusion**: Message threading errors in complex conversations

### Multi-Account Complications
- **Account Conflicts**: Same message appearing in multiple accounts
- **Authentication Mixing**: Credential confusion between accounts
- **Folder Name Collisions**: Similar folder names across accounts
- **Synchronization Issues**: Different sync states between accounts
- **Default Account Changes**: Operations using wrong account unexpectedly

### Operational and Performance Issues
- **Large Mailboxes**: Performance degradation with thousands of messages
- **Memory Consumption**: High memory usage for complex operations
- **Disk Space**: Local cache growing to consume excessive storage
- **Editor Integration**: External editor failures during message composition
- **Search Performance**: Slow search operations on large mailboxes

## Technical Debt

### Configuration and Setup Complexity
- Manual configuration file editing required for advanced setups
- Limited validation of IMAP/SMTP settings during initial configuration
- No automated discovery of email server settings
- Poor error messages for configuration problems

### User Experience Limitations
- Command-line interface intimidating for non-technical users
- No graphical interface or terminal UI mode
- Limited message preview capabilities
- Basic search interface without advanced filtering

### Performance and Scalability
- No background synchronization for large mailboxes
- Inefficient handling of very large messages
- Limited caching strategies for frequently accessed data
- No parallel processing for multi-account operations

### Integration and Extensibility
- Limited integration with external calendar and contact systems
- Basic plugin architecture without extension points
- No integration with desktop notification systems
- Missing support for modern email features (read receipts, tracking)

## Dependencies

### Core Dependencies
- **Himalaya Binary**: Primary CLI tool for email operations
- **IMAP/SMTP Libraries**: Network protocol implementations
- **MIME Parser**: Email message format processing
- **Cryptographic Libraries**: SSL/TLS and authentication support

### Authentication Dependencies
- **Password Managers**: Integration with pass, keyring systems
- **OAuth Libraries**: Modern authentication flow support
- **Command Integration**: Support for external password retrieval commands
- **Keyring Access**: System keyring integration for credential storage

### System Dependencies
- **Configuration Storage**: File system access for configuration files
- **External Editor**: Integration with $EDITOR environment variable
- **Terminal Support**: ANSI color and cursor positioning
- **Network Stack**: TCP/IP connectivity for email servers

### Optional Dependencies
- **Homebrew**: Package manager for macOS installation
- **Notmuch**: Alternative local email indexing backend
- **Sendmail**: Local SMTP delivery option
- **GPG**: Email encryption and signing support

### Development Dependencies
- **Rust Toolchain**: For building Himalaya from source
- **OpenSSL**: Cryptographic library for secure connections
- **Testing Framework**: Email server simulation for testing
- **Documentation**: Configuration guides and usage examples