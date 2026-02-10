# Gifgrep Skill Specification

## Intent

Provide comprehensive GIF search, download, and processing capabilities through CLI and TUI interfaces. Enables GIF discovery from major providers (Tenor, Giphy), local processing for still extraction, and sheet generation for documentation and sharing workflows.

## Interface

### Core Operations
- **Search**: Query GIF providers with keyword-based search
- **Browse**: Interactive TUI for GIF preview and selection
- **Download**: Fetch GIFs to local storage with organization
- **Process**: Extract stills and create frame sheets from GIFs

### Command Structure
- `gifgrep QUERY --max N` - Search and display results
- `gifgrep tui "QUERY"` - Interactive browser with previews
- `gifgrep QUERY --download --max N` - Download search results
- `gifgrep still GIF_FILE --at TIME -o OUTPUT.png` - Extract single frame
- `gifgrep sheet GIF_FILE --frames N --cols N -o OUTPUT.png` - Create frame grid

### Provider Integration
- **Tenor**: Default provider with demo API key
- **Giphy**: Requires `GIPHY_API_KEY` environment variable
- **Auto**: Intelligent provider selection based on query

### Output Formats
- **JSON**: Structured data for programmatic processing
- **URL**: Direct links for piping and automation
- **Interactive**: Rich terminal display with thumbnails

## Business Logic

### Search and Discovery Workflow
1. Accept search queries and routing to appropriate providers
2. Execute API calls with pagination and result limits
3. Parse provider responses into standardized format
4. Display results with metadata, previews, and selection options
5. Handle search refinement and filtering

### Interactive Browser Experience
1. Launch TUI with search results grid layout
2. Provide keyboard navigation and selection interface
3. Display GIF previews in terminal (when supported)
4. Enable multi-selection and batch operations
5. Integrate download and processing actions

### Download Management
1. Fetch selected GIFs from provider CDNs
2. Organize downloads in configurable directory structure
3. Handle filename conflicts and deduplication
4. Provide download progress and completion feedback
5. Optional file system integration (reveal in Finder)

### GIF Processing Pipeline
1. Load GIF files for frame extraction and analysis
2. Extract specific frames at designated timestamps
3. Generate frame sheets with configurable grid layouts
4. Apply processing options (padding, aspect ratio, sizing)
5. Export processed images in standard formats

## Data Structures

### GIF Search Result
```typescript
interface GifResult {
  id: string;                  // provider-specific GIF identifier
  title: string;               // GIF title or description
  url: string;                 // direct GIF URL for download
  previewUrl: string;          // thumbnail/preview image URL
  tags: string[];              // associated keywords and tags
  width: number;               // GIF width in pixels
  height: number;              // GIF height in pixels
  duration?: number;           // GIF duration in seconds
  size?: number;               // file size in bytes
  provider: 'tenor' | 'giphy'; // source provider
  sourceUrl?: string;          // original provider page URL
}
```

### Search Configuration
```typescript
interface SearchConfig {
  query: string;               // search keywords
  provider: 'auto' | 'tenor' | 'giphy';
  maxResults: number;          // result limit (default varies by provider)
  contentRating: string;       // safe, moderate, mature
  language?: string;           // language preference
  randomSeed?: string;         // for reproducible random results
}
```

### Processing Options
```typescript
interface ProcessingOptions {
  frameExtraction: {
    timestamp: string;         // time position (e.g., "1.5s", "50%")
    outputFormat: 'png' | 'jpg';
    quality?: number;          // JPEG quality (1-100)
  };
  sheetGeneration: {
    frameCount: number;        // number of frames to sample
    columns: number;           // grid column count
    padding: number;           // spacing between frames
    aspectRatio?: number;      // cell aspect ratio override
    maxWidth?: number;         // sheet maximum width
  };
}
```

### Download Session
```typescript
interface DownloadSession {
  sessionId: string;           // download batch identifier
  results: GifResult[];        // selected GIFs for download
  destination: string;         // target directory
  progress: DownloadProgress[];
  completed: boolean;
  errors: DownloadError[];
}

interface DownloadProgress {
  gifId: string;
  filename: string;
  bytesDownloaded: number;
  totalBytes: number;
  status: 'pending' | 'downloading' | 'completed' | 'failed';
}
```

## Constraints

### Provider API Limitations
- **Rate Limits**: Tenor and Giphy impose request frequency restrictions
- **API Keys**: Giphy requires registration for API access
- **Content Policies**: Provider-specific content filtering and restrictions
- **Geographic Availability**: Service availability varies by region

### Terminal and Display Requirements
- **TUI Compatibility**: Interactive mode requires terminal with cursor positioning
- **Preview Support**: GIF previews require compatible terminals (Kitty, Ghostty)
- **Color Support**: Rich display features require true color terminal support
- **Unicode Support**: Proper rendering of interface elements and characters

### File System and Storage
- **Download Permissions**: Write access to download directory (~/Downloads)
- **Disk Space**: Sufficient storage for downloaded GIFs and processed images
- **File Format Support**: GIF decoding and PNG/JPEG encoding capabilities
- **Path Length Limits**: File system restrictions on filename lengths

### Processing Performance
- **Memory Usage**: Large GIFs require significant RAM for processing
- **CPU Intensive**: Frame extraction and sheet generation are computationally expensive
- **File Size Limits**: Very large GIFs may cause processing failures
- **Animation Complexity**: Complex GIFs with many frames impact performance

## Edge Cases

### API and Network Issues
- **Provider Outages**: Tenor or Giphy services temporarily unavailable
- **API Key Issues**: Invalid, expired, or rate-limited API credentials
- **Network Connectivity**: Internet connectivity loss during search or download
- **Content Removed**: GIFs deleted from provider CDNs after search
- **CORS Restrictions**: Browser-like restrictions on direct GIF access

### Terminal and Display Problems
- **Unsupported Terminals**: TUI mode failing on incompatible terminals
- **Terminal Size**: Very small terminal windows breaking layout
- **Color Depth**: Limited color support affecting visual quality
- **Font Issues**: Unicode characters not displaying correctly
- **Refresh Problems**: Screen corruption during intensive TUI operations

### Download and File Management
- **Permission Denied**: Unable to write to download directory
- **Disk Full**: Insufficient storage space for downloads
- **Filename Conflicts**: Duplicate filenames requiring resolution
- **Partial Downloads**: Network interruption causing incomplete files
- **File Corruption**: Downloaded GIFs corrupted during transfer

### Processing Edge Cases
- **Invalid GIF Format**: Malformed or corrupted GIF files
- **Single Frame GIFs**: Static images misidentified as animated GIFs
- **Extreme Dimensions**: Very wide/tall GIFs causing processing issues
- **Color Palette**: Limited color palette GIFs affecting processing quality
- **Timestamp Out of Range**: Frame extraction beyond GIF duration

## Technical Debt

### Error Handling and Recovery
- Basic error messages without specific resolution guidance
- No automatic retry logic for transient network failures
- Limited graceful degradation when TUI features unavailable
- Poor error context for provider API failures

### Performance and Optimization
- No local caching of search results or metadata
- Sequential processing instead of parallel download operations
- Memory inefficient processing of large GIF files
- No progressive loading for TUI preview generation

### User Experience Limitations
- Command-line interface may intimidate non-technical users
- Limited customization options for download organization
- No bookmarking or favorites system for discovered GIFs
- Basic search refinement without advanced filtering

### Feature Completeness
- No support for video formats beyond GIF
- Limited metadata extraction and management
- No integration with popular messaging or social platforms
- Missing batch processing automation features

## Dependencies

### Core Dependencies
- **gifgrep Binary**: Primary CLI tool for GIF operations
- **HTTP Client**: For provider API communication and file downloads
- **GIF Processing**: Libraries for GIF decoding and frame extraction
- **Image Processing**: PNG/JPEG encoding for extracted frames and sheets

### Provider Dependencies
- **Tenor API**: GIF search and metadata retrieval
- **Giphy API**: Alternative GIF provider with API key requirement
- **CDN Access**: Direct download from provider content networks

### Terminal Dependencies
- **TUI Framework**: Terminal user interface rendering
- **Terminal Detection**: Capability detection for interactive features
- **Color Support**: True color and 256-color terminal support
- **Unicode Rendering**: Character encoding and display support

### System Dependencies
- **File System**: Download directory access and management
- **Network Stack**: HTTP/HTTPS connectivity for all operations
- **Process Management**: Command execution and signal handling
- **Memory Management**: Efficient handling of large binary data

### Optional Dependencies
- **Homebrew**: Package manager for macOS installation
- **Go Toolchain**: For building from source
- **Terminal Emulators**: Kitty, Ghostty for enhanced preview features
- **File Managers**: Finder integration for download reveal

### Development Dependencies
- **Go Development Tools**: Building and testing gifgrep from source
- **Image Processing Libraries**: GIF, PNG, JPEG format support
- **Testing Framework**: Automated testing of search and processing
- **Documentation Tools**: Help system and manual generation