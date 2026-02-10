# Canvas Module Specification

## Intent
The Canvas module provides dynamic web content presentation and interactive UI capabilities for OpenClaw agents. It enables real-time rendering of HTML/CSS/JavaScript content, agent-to-UI communication, screenshot capture, and A2UI (Agent-to-UI) integration for dynamic interface generation. The module serves as a visual output mechanism for agents to present rich content, dashboards, and interactive experiences.

## Interface

### Core Types
```typescript
export type CanvasAction = 'present' | 'hide' | 'navigate' | 'eval' | 'snapshot' | 
  'a2ui_push' | 'a2ui_reset';

export type CanvasRequest = {
  action: CanvasAction;
  url?: string;
  javaScript?: string;
  width?: number;
  height?: number;
  x?: number;
  y?: number;
  maxWidth?: number;
  timeoutMs?: number;
  delayMs?: number;
  outputFormat?: 'png' | 'jpg' | 'jpeg';
  quality?: number;
  node?: string;
  target?: string;
  jsonl?: string;
  jsonlPath?: string;
  gatewayUrl?: string;
  gatewayToken?: string;
};

export type CanvasWindow = {
  id: string;
  url: string;
  width: number;
  height: number;
  visible: boolean;
  position: { x: number; y: number };
  lastUpdated: Date;
};

export type A2UIComponent = {
  type: string;
  props: Record<string, any>;
  children?: A2UIComponent[];
  id?: string;
  className?: string;
};
```

### Key Functions
- `presentCanvas(url, dimensions, position)` - Display web content in overlay window
- `hideCanvas()` - Hide canvas window
- `navigateCanvas(url)` - Navigate to different URL
- `evaluateJS(script)` - Execute JavaScript in canvas context
- `captureSnapshot(options)` - Take screenshot of canvas content
- `pushA2UI(components)` - Update A2UI interface
- `resetA2UI()` - Clear A2UI content

### Canvas Actions
- `present` - Show canvas window with specified content
- `hide` - Hide canvas window from view
- `navigate` - Navigate to new URL or content
- `eval` - Execute JavaScript code in canvas context
- `snapshot` - Capture current canvas state as image
- `a2ui_push` - Push new A2UI components
- `a2ui_reset` - Reset A2UI to initial state

## Business Logic

### Canvas Lifecycle Management
1. **Window Creation**: Create canvas window with specified dimensions
2. **Content Loading**: Load HTML/CSS/JS content from URL or inline
3. **Rendering**: Render content using web engine (WebKit/Chromium)
4. **Event Handling**: Process user interactions and agent events
5. **State Management**: Maintain canvas state and history
6. **Resource Cleanup**: Proper cleanup when canvas is hidden or destroyed

### A2UI Dynamic Interface System
- **Component Definition**: Define UI components using JSON schema
- **Real-time Updates**: Push interface updates without page reload
- **Event Binding**: Connect UI events to agent functions
- **State Synchronization**: Maintain consistent state between agent and UI
- **Layout Management**: Responsive layout adaptation

### JavaScript Execution Context
- **Sandbox Security**: Execute scripts in controlled environment
- **API Exposure**: Expose safe agent APIs to canvas JavaScript
- **Error Handling**: Graceful handling of JavaScript runtime errors
- **Performance Monitoring**: Track script execution performance
- **Resource Limits**: Prevent excessive resource consumption

### Screenshot and Media Capture
- **High-Quality Rendering**: Capture canvas at various resolutions
- **Format Options**: Support multiple image formats (PNG, JPEG)
- **Selective Capture**: Capture specific regions or full canvas
- **Compression**: Optimize image size while maintaining quality
- **Batch Capture**: Support multiple simultaneous captures

### Cross-Platform Rendering
- **Platform Adaptation**: Adapt to different OS windowing systems
- **Resolution Handling**: Support high-DPI displays
- **Color Management**: Consistent color rendering across platforms
- **Performance Optimization**: Optimize rendering for different hardware
- **Accessibility**: Support accessibility features and screen readers

## Data Structures

### Canvas State
```typescript
type CanvasState = {
  windows: Map<string, CanvasWindow>;
  activeWindow?: string;
  history: HistoryEntry[];
  eventListeners: Map<string, EventCallback[]>;
  a2uiState: A2UIState;
};
```

### A2UI State
```typescript
type A2UIState = {
  components: A2UIComponent[];
  eventHandlers: Map<string, Function>;
  state: Record<string, any>;
  layout: LayoutConfig;
  theme: ThemeConfig;
};
```

### History Entry
```typescript
type HistoryEntry = {
  timestamp: Date;
  url: string;
  title?: string;
  action: CanvasAction;
  metadata: Record<string, any>;
};
```

### Event Callback
```typescript
type EventCallback = {
  id: string;
  event: string;
  handler: Function;
  once: boolean;
  context?: any;
};
```

## Constraints

### Performance Limitations
- **Memory Usage**: Canvas content limited by available system memory
- **Rendering Speed**: Complex layouts may impact rendering performance
- **JavaScript Execution**: Script execution time limits to prevent blocking
- **Image Capture**: Large screenshots require significant processing time

### Platform-Specific Constraints
- **macOS**: Uses WKWebView for rendering, limited by macOS security
- **Linux**: Requires X11 or Wayland for window management
- **Windows**: Uses WebView2, requires appropriate runtime installation
- **Mobile**: Limited canvas capabilities on mobile platforms

### Security Restrictions
- **Same-Origin Policy**: JavaScript execution subject to browser security
- **File System Access**: Limited access to local file system
- **Network Requests**: Restricted network access from canvas content
- **API Exposure**: Careful limitation of exposed agent APIs

### Resource Constraints
- **Concurrent Windows**: Maximum number of simultaneous canvas windows
- **Memory Limits**: Canvas content memory usage boundaries
- **CPU Usage**: Rendering process CPU utilization limits
- **Storage**: Temporary files and cache storage limitations

## Edge Cases

### Content Loading Failures
- **Network Errors**: Handle failed URL loading gracefully
- **Invalid Content**: Deal with malformed HTML/CSS/JS
- **Timeout Scenarios**: Handle slow-loading content appropriately
- **CORS Issues**: Navigate cross-origin request restrictions

### Window Management Edge Cases
- **Window Overlap**: Handle overlapping canvas windows
- **Screen Boundaries**: Prevent windows from going off-screen
- **Resolution Changes**: Adapt to display resolution changes
- **Multi-Monitor**: Handle multi-monitor setups appropriately

### JavaScript Runtime Issues
- **Script Errors**: Graceful handling of JavaScript exceptions
- **Infinite Loops**: Detect and terminate runaway scripts
- **Memory Leaks**: Prevent JavaScript memory leaks in long-running canvas
- **API Conflicts**: Handle conflicts between different script contexts

### A2UI State Management
- **Component Lifecycle**: Proper mounting and unmounting of components
- **State Synchronization**: Handle state inconsistencies between agent and UI
- **Event Handling**: Prevent event handler memory leaks
- **Layout Conflicts**: Resolve conflicting layout specifications

## Technical Debt

### Known Issues
- **Memory Leaks**: Canvas windows don't always clean up properly
- **Linux Compatibility**: Inconsistent behavior across Linux distributions
- **A2UI Performance**: Large component trees cause rendering lag
- **Screenshot Quality**: Inconsistent quality across different platforms

### Performance Optimizations Needed
- **Rendering Pipeline**: More efficient rendering for complex content
- **Image Compression**: Better compression algorithms for screenshots
- **Event Handling**: Optimize event propagation and handling
- **Memory Management**: Improved garbage collection for long-running canvas

### Architecture Improvements Required
- **Component System**: More robust A2UI component architecture
- **State Management**: Better state synchronization mechanisms
- **Plugin System**: Extensible plugin system for custom components
- **Theme System**: More comprehensive theming capabilities

### Platform-Specific Issues
- **macOS Permissions**: Canvas may require screen recording permissions
- **Linux X11**: Better support for different X11 configurations
- **Windows Scaling**: Inconsistent behavior with display scaling
- **Mobile Support**: Limited mobile platform support

## Dependencies

### Internal Dependencies
- **Gateway**: WebSocket communication for canvas control
- **Agents**: Integration with agent execution and tool system
- **Config**: Canvas window configuration and preferences
- **Security**: Secure script execution and API exposure

### External Dependencies
- **Web Engine**: Platform-specific web rendering engine (WebKit/Chromium)
- **Window Management**: OS windowing system for canvas display
- **Graphics Libraries**: Image processing and screenshot capabilities
- **JavaScript Runtime**: V8 or similar for script execution

### Platform Dependencies
- **macOS**: WebKit, Cocoa windowing, Core Graphics
- **Linux**: WebKit/GTK, X11/Wayland, Cairo graphics
- **Windows**: WebView2, Win32 API, Direct2D
- **Node.js**: Electron or similar for cross-platform support