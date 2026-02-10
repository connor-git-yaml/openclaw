# Things Mac Skill Specification

## Intent

Manage Things 3 task management through CLI integration on macOS, supporting task creation via URL schemes and database reading for comprehensive task and project management automation.

## Interface

### Core Operations
- **Task Management**: Add, update, and organize tasks and projects
- **Database Access**: Read and search local Things database
- **Project Organization**: Manage projects, areas, and task hierarchies
- **Due Date Handling**: Schedule tasks with dates and reminders
- **Tag Management**: Apply and organize tags for task categorization

### CLI Structure
- **Add Tasks**: `things add "Task title" --project "Work" --due today`
- **Search**: `things search "keyword" --list inbox`
- **List Views**: `things list inbox|today|upcoming|completed`
- **Projects**: `things show projects --area "Personal"`

## Business Logic

### Task Creation and URL Scheme Integration
1. Generate Things URL schemes for task and project creation
2. Handle task attributes like due dates, notes, and tags
3. Process batch task creation and template operations
4. Integrate with system notifications and reminders

### Database Reading and Search
1. Query local Things SQLite database directly
2. Search tasks by content, projects, areas, and tags
3. Generate task lists and project overviews
4. Extract task metadata and completion statistics

### Project and Area Management
1. Organize tasks within projects and areas hierarchy
2. Handle project planning and milestone tracking
3. Manage area-based task organization
4. Process project templates and recurring structures

## Data Structures

### Task Structure
```typescript
interface ThingsTask {
  uuid: string; // task identifier
  title: string; // task title
  notes?: string; // task notes
  project?: string; // parent project
  area?: string; // parent area
  dueDate?: Date; // due date
  startDate?: Date; // start date
  completionDate?: Date; // completion timestamp
  tags: string[]; // assigned tags
  status: 'open' | 'completed' | 'cancelled';
  type: 'task' | 'project' | 'heading';
}
```

### Project Information
```typescript
interface ThingsProject {
  uuid: string; // project identifier
  title: string; // project name
  notes?: string; // project description
  area?: string; // parent area
  status: 'open' | 'completed' | 'cancelled';
  tasks: ThingsTask[]; // project tasks
  dueDate?: Date; // project deadline
  tags: string[]; // project tags
}
```

## Constraints

### macOS Platform Requirements
- **Things 3 Required**: Things 3 application must be installed
- **Database Access**: Direct SQLite database file access needed
- **URL Scheme**: Things URL scheme handler must be functional
- **File Permissions**: Read access to Things database files

### Database and Sync Limitations
- **Local Database**: Only accesses local database, not cloud sync state
- **Read-Only Queries**: Database modifications through CLI are limited
- **Sync Conflicts**: Direct database access may conflict with app sync
- **Schema Changes**: Things app updates may change database schema

## Edge Cases

### Application and Database Issues
- **Things Not Installed**: Things 3 application not available
- **Database Locked**: Things app using database during CLI access
- **Corrupted Database**: Database file corruption or inaccessibility
- **Permission Denied**: Insufficient file system permissions
- **Schema Migration**: Database schema changes after Things updates

### URL Scheme and Integration Problems
- **URL Handler Failed**: Things not responding to URL scheme calls
- **Malformed URLs**: Invalid URL scheme syntax causing failures
- **App Not Running**: Things 3 not launched for URL handling
- **Security Restrictions**: System blocking URL scheme execution

## Technical Debt

### Database Access Complexity
- Direct SQLite access fragility with app updates
- Limited abstraction layer for database schema changes
- No proper conflict resolution with Things app operations

### URL Scheme Limitations
- Limited task attributes supported via URL schemes
- No bulk operations or complex task relationships
- Error handling complexity for failed URL operations

## Dependencies

### Core Dependencies
- **things CLI**: Things 3 command-line interface tool
- **Things 3 App**: Things 3 macOS application installation
- **SQLite**: Database access for reading Things data
- **URL Scheme Handler**: macOS URL scheme processing

### System Dependencies
- **macOS Platform**: Operating system compatibility
- **File System**: Access to Things database files
- **Application Framework**: macOS app integration capabilities
- **Shell Environment**: Command execution environment