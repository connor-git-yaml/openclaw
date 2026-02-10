# Bear Notes Skill Specification

## Intent

Enable command-line interface to Bear note-taking app on macOS through the `grizzly` CLI tool. Provides note creation, reading, editing, tagging, and search functionality for Bear users who prefer terminal-based workflows or need programmatic note management.

## Interface

### Primary Operations
- **Note Creation**: Create new notes with title, content, and tags
- **Note Reading**: Access note content by ID with JSON output
- **Note Editing**: Append/prepend text to existing notes
- **Tag Management**: List, search, and organize notes by tags
- **Search & Discovery**: Find notes through tag-based filtering

### Command Patterns
- `grizzly create --title "..." --tag TAG` - Create new note with metadata
- `grizzly open-note --id ID --enable-callback --json` - Read note content
- `grizzly add-text --id ID --mode append` - Edit existing note
- `grizzly tags --enable-callback --json` - List all available tags
- `grizzly open-tag --name TAG --enable-callback --json` - Search by tag

### Authentication Methods
- Token-based authentication for advanced operations
- Token storage in `~/.config/grizzly/token` file
- Optional operations work without authentication

## Business Logic

### Authentication Flow
1. Generate Bear API token from Bear app (Help → API Token)
2. Store token in configuration file or environment variable
3. Validate token for operations requiring authentication
4. Handle token expiration and renewal

### Note Creation Workflow
1. Accept note content via stdin or direct parameters
2. Apply title and tag metadata to new note
3. Execute Bear x-callback-url for note creation
4. Return note ID for subsequent operations

### Callback Mechanism
1. Enable callback URLs for operations requiring data return
2. Wait for Bear app response with timeout handling
3. Parse JSON responses for note content and metadata
4. Handle callback failures and retry logic

### Configuration Management
1. Load configuration from multiple sources (CLI, env, files)
2. Priority order: CLI flags → environment → local config → global config
3. Validate configuration before executing operations

## Data Structures

### Note Entity
```typescript
interface BearNote {
  id: string;                  // Bear internal note identifier
  title: string;               // note title
  content: string;             // full note content (Markdown)
  tags: string[];              // associated tag list
  created: Date;               // creation timestamp
  modified: Date;              // last modification timestamp
  isArchived: boolean;         // archive status
}
```

### Tag Structure
```typescript
interface BearTag {
  name: string;                // tag name (without #)
  count: number;               // number of notes with tag
  isNested: boolean;           // whether tag has subtags
  parent?: string;             // parent tag for nested structure
}
```

### Configuration
```typescript
interface GrizzlyConfig {
  tokenFile: string;           // path to Bear API token
  callbackUrl: string;         // callback URL for responses
  timeout: string;             // operation timeout duration
  dryRun: boolean;             // preview mode flag
  enableJson: boolean;         // JSON output preference
}
```

### Operation Result
```typescript
interface GrizzlyResult {
  success: boolean;
  noteId?: string;             // for create operations
  callbackData?: any;          // JSON response from Bear
  error?: string;              // error description
  bearUrl?: string;            // x-callback-url executed
}
```

## Constraints

### Application Dependencies
- **Bear App Required**: Bear must be installed and running on macOS
- **Token Limitations**: Advanced operations require valid API token
- **Callback Dependencies**: Data retrieval requires callback URL handling
- **Platform Restriction**: macOS-only due to Bear app availability

### Technical Limitations
- **Note ID Requirements**: Most operations need Bear's internal note IDs
- **Timeout Constraints**: Callback operations subject to timeout limits
- **Bear App State**: Operations fail if Bear app is not responding
- **Network Dependency**: Callback mechanism requires local network access

### Authentication Scope
- **Token-based Operations**: add-text, tags, selected note access require token
- **Public Operations**: create, open-note by ID work without authentication
- **Token Persistence**: Manual token management required
- **Token Security**: Clear-text token storage in configuration files

## Edge Cases

### Application State Issues
- **Bear Not Running**: Operations fail when Bear app is closed
- **Bear Unresponsive**: App hanging during callback operations
- **App Version Conflicts**: grizzly compatibility with Bear updates
- **Concurrent Access**: Multiple grizzly instances accessing Bear simultaneously

### Token Management Problems
- **Invalid Token**: Expired or malformed API token
- **Missing Token File**: Token file doesn't exist for authenticated operations
- **Permission Issues**: Unable to read/write token configuration files
- **Token Rotation**: Bear regenerates token invalidating stored version

### Note Reference Failures
- **Invalid Note IDs**: References to deleted or non-existent notes
- **Note Access Denied**: Private or protected notes inaccessible
- **Content Corruption**: Notes with malformed or corrupted content
- **Large Notes**: Performance issues with very large note content

### Network and Callback Issues
- **Callback Timeout**: Bear response exceeds configured timeout
- **Port Conflicts**: Callback URL port already in use
- **Network Blocking**: Firewall blocking localhost callback communication
- **Malformed Responses**: Invalid JSON in Bear callback responses

## Technical Debt

### Error Handling Deficiencies
- Limited error context for Bear app communication failures
- No graceful degradation when callback mechanism unavailable
- Insufficient validation of note IDs before operations
- Poor error messages for authentication failures

### Configuration Complexity
- Multiple configuration sources create confusion
- No configuration validation at startup
- Environment variable naming inconsistencies
- Missing configuration migration tools

### Performance Limitations
- No caching of note metadata between operations
- Inefficient tag listing for large note collections
- No bulk operations for multiple notes
- Sequential processing of batch operations

### Feature Gaps
- Missing support for Bear's advanced features (sketches, files)
- No note export functionality to other formats
- Limited search capabilities compared to Bear app
- No support for Bear's collaborative features

## Dependencies

### Core Dependencies
- **grizzly CLI**: Primary interface binary for Bear integration
- **Bear App**: Target application for note storage and management
- **Go Runtime**: Required for grizzly binary execution

### Installation Dependencies
- **Go Toolchain**: For installing grizzly from source
- **Git**: For source code retrieval during installation
- **Network Access**: For downloading grizzly module

### System Dependencies
- **macOS**: Operating system requirement for Bear app
- **x-callback-url Support**: System-level URL scheme handling
- **Network Stack**: Localhost communication for callbacks

### Configuration Dependencies
- **File System**: For configuration and token storage
- **Environment Variables**: For runtime configuration override
- **TOML Parser**: For configuration file processing

### Optional Dependencies
- **Bear Pro**: For advanced Bear features (if used in notes)
- **Markdown Processor**: For note content formatting
- **JSON Parser**: For callback response processing
- **HTTP Server**: For callback URL handling

### Development Dependencies
- **Go Development Tools**: For building grizzly from source
- **Git**: For version control and source management
- **Testing Framework**: For grizzly functionality testing