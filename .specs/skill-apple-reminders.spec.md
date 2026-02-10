# Apple Reminders Skill Specification

## Intent

Provide command-line access to Apple Reminders.app on macOS via the `remindctl` CLI tool. Enables complete reminder management including creation, editing, completion, deletion, and advanced filtering by dates, lists, and status for personal productivity workflows.

## Interface

### Core Operations
- **Reminder Management**: Add, edit, complete, delete individual reminders
- **List Organization**: Create, rename, delete reminder lists
- **Time-based Views**: Today, tomorrow, week, overdue, upcoming filters
- **Status Management**: Complete/incomplete reminder state changes

### Command Structure
- `remindctl [date-filter]` - View reminders by time period
- `remindctl add --title "..." --list "..." --due DATE` - Create reminders
- `remindctl edit ID --title "..." --due DATE` - Modify existing reminders
- `remindctl complete ID [ID...]` - Mark reminders complete
- `remindctl delete ID --force` - Remove reminders permanently
- `remindctl list [NAME] [--create|--rename|--delete]` - List management

### Output Formats
- Human-readable default format with colors and icons
- JSON format (`--json`) for programmatic access
- Plain TSV format (`--plain`) for simple parsing
- Count-only format (`--quiet`) for statistics

## Business Logic

### Permission Management
1. Check Reminders.app access status via `remindctl status`
2. Request authorization using `remindctl authorize` if needed
3. Handle permission denial with guidance to System Settings
4. Validate permissions before executing operations

### Date Processing
1. Parse flexible date inputs (relative, absolute, ISO 8601)
2. Convert to Apple Reminders.app internal date format
3. Handle timezone considerations for due date calculations
4. Support both date-only and date-time specifications

### List Operations
1. Enumerate available reminder lists
2. Create new lists with unique naming
3. Handle list renaming with conflict resolution
4. Manage list deletion with reminder preservation/migration

### Reminder Lifecycle
1. Create reminders with title, list assignment, due date
2. Track reminder IDs for subsequent operations
3. Update reminder properties atomically
4. Handle completion state changes with timestamp tracking

## Data Structures

### Reminder Entity
```typescript
interface AppleReminder {
  id: string;                  // unique reminder identifier
  title: string;               // reminder text content
  list: string;                // containing list name
  dueDate?: Date;              // optional due date/time
  completed: boolean;          // completion status
  completedDate?: Date;        // completion timestamp
  notes?: string;              // additional reminder notes
}
```

### Reminder List
```typescript
interface ReminderList {
  name: string;                // list display name
  id: string;                  // internal list identifier
  reminderCount: number;       // total reminders in list
  completedCount: number;      // completed reminders count
  color?: string;              // list color designation
}
```

### Date Filter
```typescript
interface DateFilter {
  type: 'today' | 'tomorrow' | 'week' | 'overdue' | 'upcoming' | 'completed' | 'all';
  specificDate?: string;       // YYYY-MM-DD format
  includeCompleted: boolean;   // show completed reminders
}
```

### Operation Result
```typescript
interface RemindctlResult {
  success: boolean;
  remindersAffected: number;
  newReminderId?: string;      // for add operations
  error?: string;              // error description
}
```

## Constraints

### Platform Dependencies
- **macOS Only**: Requires Apple Reminders.app and EventKit framework
- **Reminders.app**: Must be installed and accessible on system
- **Authorization**: Requires explicit user permission for Reminders access

### Command Limitations
- **ID Requirements**: Edit/complete/delete operations require reminder IDs
- **Force Deletion**: Delete operations must include --force flag for safety
- **List Conflicts**: Cannot create duplicate list names
- **Date Parsing**: Limited to specific date format patterns

### Permission Scope
- **Terminal Access**: Must grant permission to specific terminal application
- **SSH Limitations**: Remote access requires permission on local Mac
- **Background Access**: Some operations may require Reminders.app to be running

## Edge Cases

### Permission Failures
- **Access Denied**: User denies Reminders permission request
- **Permission Revoked**: Previously granted access removed in System Settings
- **Terminal Switching**: Different terminals require separate permissions
- **SSH Context**: Remote execution without local permission grants

### Data Inconsistencies
- **Missing Reminders**: IDs reference deleted or corrupted reminders
- **List Conflicts**: Operations on non-existent or renamed lists
- **Sync Issues**: iCloud synchronization creating duplicate reminders
- **Concurrent Modifications**: Reminders.app changes during CLI operations

### Date/Time Issues
- **Invalid Dates**: Malformed date strings in --due parameter
- **Timezone Confusion**: Local vs UTC time interpretation
- **Past Due Dates**: Setting due dates in the past
- **Daylight Saving**: Time zone transition edge cases

### System State Problems
- **Reminders.app Crashes**: Target application unavailable during operation
- **Database Corruption**: EventKit database integrity issues
- **Memory Constraints**: Large reminder datasets causing performance issues
- **Network Dependencies**: iCloud sync failures affecting local operations

## Technical Debt

### Error Handling Gaps
- Insufficient validation of reminder IDs before operations
- Limited feedback for permission-related failures
- No retry mechanisms for transient EventKit errors
- Unclear error messages for complex date parsing failures

### Performance Considerations
- No pagination for large reminder datasets
- Inefficient querying when filtering across multiple lists
- Memory usage scales poorly with reminder count
- No caching of list metadata between operations

### Feature Incompleteness
- Missing support for reminder priorities/flags
- No attachment or note management capabilities
- Limited recurring reminder support
- No bulk operations for multiple reminders

### Integration Limitations
- No notification integration with system alerts
- Limited calendar app integration
- No support for shared reminder lists
- Missing location-based reminder features

## Dependencies

### Core Dependencies
- **remindctl**: Primary CLI binary for Reminders.app access
- **Apple Reminders.app**: Target application for reminder storage
- **EventKit Framework**: macOS framework for calendar/reminder data access

### Installation Dependencies
- **Homebrew**: Package manager for remindctl installation
- **steipete/tap**: Third-party Homebrew tap for remindctl
- **Node.js/pnpm**: Alternative build tools for source installation

### System Dependencies
- **macOS**: Operating system platform requirement
- **System Permissions**: Privacy & Security framework for app authorization
- **EventKit Database**: System-level reminder data storage

### Runtime Dependencies
- **Terminal Application**: Command execution environment
- **Shell Environment**: Command parsing and output formatting
- **Standard I/O**: Input/output stream handling for interactive operations

### Optional Dependencies
- **iCloud**: For reminder synchronization across devices
- **Calendar.app**: For integrated calendar/reminder workflows
- **Notification Center**: For system-level reminder alerts
- **Shortcuts.app**: For automation workflow integration

### Development Dependencies
- **TypeScript**: Language for remindctl source code
- **Node.js Build Tools**: For compilation and packaging
- **Git**: Source control for installation from repository