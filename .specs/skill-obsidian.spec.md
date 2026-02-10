# Obsidian Skill Specification

## Intent

Provide comprehensive Obsidian vault integration for managing plain Markdown notes through obsidian-cli automation. Enables vault discovery, note creation/editing, search operations, link management, and workspace automation for knowledge management workflows.

## Interface

### Core Operations
- **Vault Management**: Discover and switch between multiple Obsidian vaults
- **Note Operations**: Create, read, update, delete, and organize Markdown notes
- **Search Functionality**: Search note names and content with fuzzy matching
- **Link Management**: Maintain wikilinks and markdown links across vault operations
- **Workspace Automation**: Trigger Obsidian UI operations via URI handler

### CLI Structure
- **Vault Config**: `print-default`, `set-default` - Vault selection and configuration
- **Search**: `search "query"`, `search-content "query"` - Note discovery operations
- **Note Management**: `create`, `move`, `delete` - File lifecycle operations
- **Direct File Access**: Standard file operations on `.md` files in vault directories

### Vault Discovery
- **Config Location**: `~/Library/Application Support/obsidian/obsidian.json`
- **Active Vault**: Vault with `"open": true` property
- **Vault Structure**: Standard folder containing `.md` files and `.obsidian/` config

### File Patterns
- Notes: `*.md` (plain text Markdown)
- Canvases: `*.canvas` (JSON format)
- Config: `.obsidian/` directory (workspace and plugin settings)
- Attachments: User-configured folder for media files

## Business Logic

### Vault Discovery and Management
1. Parse Obsidian config file to identify available vaults
2. Determine active vault from configuration or user default
3. Validate vault structure and accessibility
4. Handle multiple vault scenarios and switching

### Note Lifecycle Operations
1. Create notes with proper path structure and optional content
2. Move/rename notes while preserving link integrity
3. Delete notes with dependency checking
4. Validate note paths and avoid hidden directories

### Search and Discovery
1. Execute fuzzy search across note names and metadata
2. Perform content search with snippet extraction
3. Handle search result ranking and filtering
4. Process search queries with proper escaping

### Link Integrity Management
1. Update wikilinks `[[note]]` during move operations
2. Maintain markdown links during file operations
3. Detect and resolve broken links
4. Handle cross-vault link scenarios

### URI Handler Integration
1. Generate proper `obsidian://` URIs for UI operations
2. Validate Obsidian application availability
3. Handle URI scheme registration and permissions
4. Trigger note opening and workspace navigation

## Data Structures

### Vault Configuration
```typescript
interface VaultConfig {
  path: string;                // full filesystem path to vault
  ts: number;                  // timestamp
  open: boolean;               // vault active status
  id: string;                  // unique vault identifier
}

interface ObsidianConfig {
  vaults: Record<string, VaultConfig>; // vault name -> config mapping
  insider: boolean;            // Obsidian Insider version flag
}
```

### Note Structure
```typescript
interface ObsidianNote {
  path: string;                // relative path within vault
  name: string;                // note title/filename
  content: string;             // markdown content
  frontmatter?: Record<string, any>; // YAML metadata
  created: Date;               // file creation time
  modified: Date;              // last modification time
  size: number;                // file size in bytes
}
```

### Search Results
```typescript
interface SearchResult {
  type: 'note' | 'content';    // search result type
  path: string;                // relative path within vault
  name: string;                // note name
  matches: SearchMatch[];      // search hit details
}

interface SearchMatch {
  line: number;                // line number (for content search)
  column: number;              // column position
  snippet: string;             // surrounding context
  highlight: string;           // matched text
}
```

### Link Reference
```typescript
interface LinkReference {
  type: 'wikilink' | 'markdown'; // link format type
  source: string;              // source note path
  target: string;              // target note path
  display?: string;            // display text
  line: number;                // line number in source
  valid: boolean;              // link target exists
}
```

### Vault Operations
```typescript
interface VaultOperation {
  type: 'create' | 'move' | 'delete' | 'search';
  target: string;              // target note path
  source?: string;             // source path (for move)
  content?: string;            // note content (for create)
  options: {
    open?: boolean;            // open in Obsidian UI
    updateLinks?: boolean;     // maintain link integrity
    force?: boolean;           // override safety checks
  };
}
```

### CLI Configuration
```typescript
interface ObsidianCLIConfig {
  defaultVault?: string;       // user-selected default vault
  vaultPath: string;           // resolved vault filesystem path
  obsidianPath: string;        // Obsidian application path
  uriScheme: 'obsidian://';    // URI scheme for app integration
}
```

## Constraints

### File System Limitations
- **Vault Structure**: Must maintain valid Obsidian vault structure
- **Path Restrictions**: Cannot create notes in hidden directories via URI
- **File Naming**: Markdown filename conventions and character limitations
- **Concurrent Access**: File locking during Obsidian application usage

### Obsidian Application Dependencies
- **Installation Required**: Obsidian desktop application must be installed
- **URI Handler**: Obsidian URI scheme must be registered
- **Version Compatibility**: obsidian-cli compatibility with Obsidian versions
- **Platform Differences**: macOS-specific config locations

### Link Management Constraints
- **Link Formats**: Limited to wikilinks and standard markdown links
- **Cross-Vault**: Links across vaults not automatically maintained
- **Embedded Files**: Attachment links may require special handling
- **Plugin Dependencies**: Some link types depend on specific plugins

### Performance and Scale
- **Large Vaults**: Performance degradation with many notes
- **Search Indexing**: No built-in search indexing for large content
- **File Operations**: Sequential operations for bulk changes
- **Memory Usage**: Content search memory requirements for large files

## Edge Cases

### Vault Configuration Issues
- **Missing Config**: Obsidian config file not found or corrupted
- **No Active Vault**: No vault marked as open in configuration
- **Multiple Active**: Multiple vaults marked as active simultaneously
- **Vault Moved**: Configured vault path no longer exists
- **Permission Issues**: Vault directory not readable/writable

### File System Conflicts
- **Name Collisions**: Creating notes with existing names
- **Invalid Paths**: Special characters or reserved names in paths
- **Case Sensitivity**: File system case handling differences
- **Long Paths**: Path length limitations on different platforms
- **Locked Files**: Files in use by Obsidian or other applications

### Link Integrity Failures
- **Broken References**: Links pointing to non-existent notes
- **Circular References**: Note rename operations creating cycles
- **Partial Updates**: Link updates failing partway through operation
- **Plugin Conflicts**: Custom link formats not recognized
- **Encoding Issues**: Unicode characters in link targets

### Application Integration Problems
- **Obsidian Not Running**: URI operations when app is closed
- **URI Handler Failure**: System URI handler not properly configured
- **Version Mismatches**: obsidian-cli incompatible with Obsidian version
- **Sandbox Restrictions**: Security restrictions preventing URI operations
- **Background Operation**: App switching behavior during automation

## Technical Debt

### CLI Tool Dependency
- External dependency on obsidian-cli tool availability and maintenance
- Limited error handling and recovery options from CLI failures
- No built-in retry mechanisms for transient CLI errors
- Version compatibility management between tool and Obsidian

### Configuration Management Complexity
- Manual parsing of Obsidian configuration files
- No official API for vault discovery and management
- Platform-specific configuration locations and formats
- No automatic vault configuration validation

### Limited Bulk Operations
- No native support for bulk note operations
- Sequential processing for multi-note workflows
- No transaction support for atomic operations
- Limited progress tracking for long-running operations

### Search and Indexing Limitations
- No persistent search index for large vaults
- Limited search query syntax and filtering options
- No full-text search optimization for performance
- Memory-intensive content search operations

## Dependencies

### Core Dependencies
- **obsidian-cli**: Third-party CLI tool for Obsidian automation
- **Obsidian Desktop**: Obsidian application for URI handler support
- **File System**: Standard file operations for vault access
- **JSON Parser**: Configuration file parsing and manipulation

### System Dependencies
- **Operating System**: macOS for standard configuration locations
- **URI Handler**: System URI scheme registration for obsidian://
- **Shell Environment**: Command execution environment for CLI operations
- **File Permissions**: Read/write access to vault directories

### Development Dependencies
- **Homebrew**: Package manager for obsidian-cli installation
- **Markdown Parser**: Content processing and validation
- **Text Processing**: Search and link extraction utilities
- **Path Utilities**: Cross-platform path manipulation

### Optional Dependencies
- **Git Integration**: Version control for note history
- **Sync Services**: iCloud or other cloud storage for vault sync
- **Backup Tools**: Vault backup and recovery systems
- **Plugin Ecosystem**: Obsidian plugins for extended functionality

### Security Dependencies
- **Sandbox Permissions**: File system and URI handler permissions
- **Code Signing**: Trusted execution of obsidian-cli tool
- **Data Privacy**: Secure handling of note content and metadata
- **Access Control**: Vault-level permissions and sharing restrictions

### Performance Dependencies
- **Search Indexing**: External search tools for large vault performance
- **Caching Layer**: File metadata caching for improved responsiveness
- **Batch Processing**: Efficient bulk operation utilities
- **Memory Management**: Large file handling and memory optimization