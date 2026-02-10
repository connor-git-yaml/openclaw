# Blogwatcher Skill Specification

## Intent

Provide command-line interface for monitoring blogs and RSS/Atom feeds with persistent tracking of read/unread status. Enables efficient content consumption workflows for users who follow multiple blogs and need organized access to new articles.

## Interface

### Core Operations
- **Feed Management**: Add, remove, and list tracked blogs/RSS feeds
- **Content Discovery**: Scan feeds for new articles and updates
- **Reading Tracking**: Mark articles as read/unread with persistent state
- **Article Listing**: View available articles with filtering options

### Command Structure
- `blogwatcher add NAME URL` - Add blog/feed to tracking list
- `blogwatcher blogs` - List all tracked blogs with status
- `blogwatcher scan` - Check all feeds for new articles
- `blogwatcher articles` - Display available articles
- `blogwatcher read ID` - Mark specific article as read
- `blogwatcher read-all` - Mark all articles as read
- `blogwatcher remove NAME` - Stop tracking specific blog

### Output Patterns
- Tabular listing of blogs with metadata
- Article summaries with read/unread indicators
- Scan progress with new article counts
- Structured feed information display

## Business Logic

### Feed Detection and Processing
1. Accept blog URLs and attempt auto-discovery of RSS/Atom feeds
2. Parse feed metadata (title, description, last update)
3. Extract article entries with titles, links, dates, and content
4. Store feed structure and article metadata locally

### Update Scanning Workflow
1. Iterate through all tracked blogs
2. Fetch latest feed content via HTTP
3. Compare against stored article database
4. Identify new articles since last scan
5. Update local database with new entries
6. Report scan results with counts

### Read State Management
1. Maintain persistent database of article read/unread status
2. Track read timestamps for historical analysis
3. Provide bulk operations for read state changes
4. Support read state synchronization across scans

## Data Structures

### Blog Entry
```typescript
interface TrackedBlog {
  name: string;                // user-assigned blog name
  url: string;                 // original blog URL
  feedUrl: string;             // discovered RSS/Atom feed URL
  title?: string;              // feed title from metadata
  description?: string;        // feed description
  lastScan: Date;              // timestamp of last scan
  articleCount: number;        // total articles discovered
}
```

### Article Entry
```typescript
interface BlogArticle {
  id: number;                  // unique article identifier
  blogName: string;            // source blog name
  title: string;               // article title
  url: string;                 // article permalink
  published: Date;             // article publication date
  content?: string;            // article content/summary
  isRead: boolean;             // read status flag
  readAt?: Date;               // read timestamp
}
```

### Scan Result
```typescript
interface ScanResult {
  blogName: string;
  sourceType: 'RSS' | 'Atom';  // feed format detected
  totalFound: number;          // total articles in feed
  newArticles: number;         // new articles since last scan
  errors?: string[];           // scan error messages
}
```

### Database State
```typescript
interface BlogwatcherState {
  blogs: TrackedBlog[];
  articles: BlogArticle[];
  lastGlobalScan: Date;
  version: string;             // database schema version
}
```

## Constraints

### Feed Compatibility
- **Format Support**: Limited to RSS 2.0 and Atom 1.0 feed formats
- **Auto-discovery**: May fail for blogs without proper feed metadata
- **Content Parsing**: Complex HTML content may not parse correctly
- **Encoding Issues**: Non-UTF8 feeds may cause parsing errors

### Network Dependencies
- **Internet Connectivity**: Requires network access for feed scanning
- **HTTP Limitations**: No support for authenticated feeds
- **Rate Limiting**: No built-in rate limiting for feed requests
- **Timeout Handling**: Long feed responses may cause scan failures

### Storage Limitations
- **Local Database**: All data stored locally without cloud sync
- **Disk Space**: Large feeds consume increasing storage over time
- **Backup**: No automated backup of tracking database
- **Migration**: Limited tools for data export/import

## Edge Cases

### Feed Access Problems
- **Dead Links**: Blogs that have moved or disappeared
- **Feed Changes**: RSS URLs that change without notification
- **Authentication**: Protected feeds requiring login credentials
- **Malformed Feeds**: Invalid XML/syntax in RSS/Atom content
- **Rate Limiting**: Servers blocking frequent feed requests

### Content Processing Issues
- **Large Feeds**: Feeds with hundreds of articles affecting performance
- **Duplicate Articles**: Same content published with different URLs
- **Date Parsing**: Inconsistent date formats in article metadata
- **Encoding Problems**: Special characters causing display issues
- **Partial Content**: Feeds providing only article summaries vs full text

### Database Consistency
- **Concurrent Access**: Multiple blogwatcher instances modifying state
- **Corruption Recovery**: Database corruption from unexpected shutdowns
- **Schema Changes**: Updates requiring database migration
- **Disk Full**: Storage exhaustion preventing new article storage

### Network and Performance
- **Slow Feeds**: Long response times during scan operations
- **Bandwidth Limits**: High-frequency scanning consuming network resources
- **DNS Issues**: Temporary domain resolution failures
- **SSL Errors**: Certificate problems with HTTPS feeds

## Technical Debt

### Error Handling Gaps
- Limited retry logic for transient network failures
- Poor error reporting for feed parsing issues
- No graceful handling of partial scan failures
- Insufficient validation of URL inputs

### Performance Limitations
- Sequential feed processing instead of parallel scanning
- No incremental updates for large feed histories
- Memory usage scales with article database size
- No cleanup of old articles from storage

### Feature Completeness
- Missing search functionality across articles
- No categorization or tagging system for blogs
- Limited export options for article data
- No notification system for new articles

### User Experience Issues
- Command-line interface may be intimidating for non-technical users
- No interactive mode for bulk operations
- Limited configuration options for scan behavior
- No progress indicators for long-running operations

## Dependencies

### Core Dependencies
- **blogwatcher Binary**: Primary CLI tool for feed management
- **Go Runtime**: Required for blogwatcher execution
- **HTTP Client**: For feed fetching and content retrieval

### Network Dependencies
- **Internet Connectivity**: Required for all feed operations
- **DNS Resolution**: For blog URL to IP address translation
- **HTTP/HTTPS**: Protocol support for feed access
- **TLS/SSL**: Secure connection handling for HTTPS feeds

### Storage Dependencies
- **Local File System**: For article database and configuration storage
- **SQLite/Database**: Local storage backend for persistent state
- **File Permissions**: Read/write access to blogwatcher data directory

### Development Dependencies
- **Go Toolchain**: For building blogwatcher from source
- **Git**: For source code management and updates
- **Build Tools**: For compilation and packaging

### Optional Dependencies
- **Cron/Scheduler**: For automated periodic feed scanning
- **Notification System**: For alerting on new articles
- **Text Processing**: For article content manipulation
- **Export Tools**: For data backup and migration