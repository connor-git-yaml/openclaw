# Apple Notes Skill Specification

## Intent

Provide terminal-based interface to Apple Notes.app on macOS through the `memo` CLI tool. Enables note management operations including creation, viewing, editing, deletion, search, organization by folders, and export functionality for personal note-taking workflows.

## Interface

### Primary Operations
- **Note Management**: Create, view, edit, delete notes with interactive prompts
- **Search & Discovery**: Fuzzy search across note content, filter by folders
- **Organization**: Move notes between folders, list folder contents
- **Export**: Convert notes to HTML/Markdown formats for external use

### Command Patterns
- `memo notes` - List all notes or filter by criteria
- `memo notes -a [title]` - Create new note (interactive or with title)
- `memo notes -e` - Edit existing note (interactive selection)
- `memo notes -d` - Delete note (interactive selection)
- `memo notes -m` - Move note between folders (interactive)
- `memo notes -ex` - Export note to HTML/Markdown

### User Experience
- Interactive CLI prompts for note selection
- System editor integration for note composition
- Fuzzy search for quick note discovery
- Folder-based organization support

## Business Logic

### Note Creation Flow
1. Invoke `memo notes -a` with optional title parameter
2. Launch system editor for note composition
3. Save note content to Apple Notes.app database
4. Return creation confirmation or error

### Search & Retrieval
1. Execute fuzzy search against note content and titles
2. Apply folder filters if specified (`-f "Folder Name"`)
3. Present interactive selection menu for results
4. Display note content or perform requested action

### Cross-Application Integration
1. Interface with Apple Notes.app via macOS automation APIs
2. Respect Notes.app folder structure and permissions
3. Handle authorization requests for Notes.app access
4. Sync operations with Notes.app state changes

## Data Structures

### Note Entity
```typescript
interface AppleNote {
  title: string;               // note title/first line
  content: string;             // full note content
  folder: string;              // containing folder name
  created: Date;               // creation timestamp
  modified: Date;              // last modification timestamp
  hasAttachments: boolean;     // contains images/files
}
```

### Folder Structure
```typescript
interface NotesFolder {
  name: string;                // folder display name
  noteCount: number;           // number of notes in folder
  isSystem: boolean;           // system vs user folder
}
```

### Search Query
```typescript
interface SearchParams {
  query?: string;              // fuzzy search text
  folder?: string;             // folder filter
  includeContent: boolean;     // search content vs titles only
}
```

### Export Options
```typescript
interface ExportConfig {
  format: 'html' | 'markdown'; // output format
  includeMeta: boolean;        // include timestamps/metadata
  outputPath?: string;         // destination file path
}
```

## Constraints

### Platform Requirements
- **macOS Only**: Requires Apple Notes.app and macOS automation APIs
- **Notes.app Dependency**: Must have Apple Notes installed and accessible
- **Automation Permissions**: Requires System Settings > Privacy & Security > Automation approval

### Content Limitations
- **Rich Media**: Cannot edit notes containing images or attachments
- **Formatting**: Limited rich text editing capabilities
- **Sync Dependencies**: Operations may fail if iCloud sync is active

### Technical Dependencies
- **memo CLI**: Requires antoniorodr/memo via Homebrew tap
- **System Editor**: Relies on $EDITOR or system default for note composition
- **Python Runtime**: memo tool requires Python execution environment

## Edge Cases

### Permission Issues
- **Automation Denied**: macOS blocks Notes.app automation access
- **Notes.app Unavailable**: Application not installed or corrupted
- **Folder Access**: Insufficient permissions for specific note folders

### Content Conflicts
- **Rich Notes**: Attempting to edit notes with attachments/images
- **Large Notes**: Performance issues with very large note content
- **Concurrent Edits**: Notes modified in Notes.app while CLI editing
- **Unicode Issues**: Special characters in note titles/content

### Sync State Problems
- **iCloud Sync**: Notes unavailable during cloud synchronization
- **Network Issues**: Cloud-synced notes inaccessible offline
- **Conflicted States**: Notes with sync conflicts between devices

### Interactive Failures
- **No TTY**: Interactive prompts fail in non-terminal environments
- **Selection Timeout**: User abandons interactive selection prompts
- **Editor Crashes**: System editor fails during note composition

## Technical Debt

### Automation Framework Dependency
- Relies on macOS automation which Apple may restrict in future versions
- No fallback mechanism if automation access is revoked
- Limited error handling for permission edge cases

### Interactive Design Limitations
- Heavy reliance on interactive prompts reduces automation potential
- No batch operation support for multiple notes
- Command-line arguments insufficient for advanced workflows

### Export Feature Gaps
- Limited export format options (only HTML/Markdown)
- No bulk export functionality
- Export metadata incomplete compared to Notes.app native export

### Error Recovery
- Insufficient handling of Notes.app state changes during operations
- No transaction rollback for failed multi-step operations
- Limited logging for troubleshooting automation issues

## Dependencies

### Core Dependencies
- **memo CLI**: Primary interface to Apple Notes functionality
- **Apple Notes.app**: Target application for note storage/management
- **macOS Automation**: System-level integration framework

### Installation Dependencies
- **Homebrew**: Package manager for memo installation
- **antoniorodr/memo Tap**: Third-party Homebrew tap
- **Python**: Runtime environment for memo tool execution

### System Dependencies
- **macOS**: Operating system platform requirement
- **System Editor**: Text editor for note composition ($EDITOR)
- **Automation Permissions**: macOS privacy/security framework

### Optional Dependencies
- **iCloud**: For note synchronization across devices
- **Mistune**: Markdown processing library for exports
- **Terminal Emulator**: For interactive command execution

### Development Dependencies
- **Git**: For memo source code management (manual installation)
- **pip**: Alternative installation method from source
- **Python Development Tools**: For building from source