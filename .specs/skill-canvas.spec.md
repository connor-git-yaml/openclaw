# Canvas Skill Specification

## Intent

Enable presentation and control of HTML content on connected OpenClaw nodes (Mac, iOS, Android) through a distributed canvas system. Supports interactive visualizations, dashboards, games, and dynamic web content with real-time updates and cross-platform rendering.

## Interface

### Core Actions
- **present**: Display canvas with HTML content on target node
- **hide**: Hide canvas view on node
- **navigate**: Change canvas content to new URL
- **eval**: Execute JavaScript code within canvas context
- **snapshot**: Capture screenshot of current canvas content

### Command Structure
- `canvas action:present node:NODE_ID target:URL` - Show content
- `canvas action:navigate node:NODE_ID url:URL` - Navigate to new content
- `canvas action:eval node:NODE_ID javaScript:CODE` - Run JavaScript
- `canvas action:snapshot node:NODE_ID` - Take screenshot
- `canvas action:hide node:NODE_ID` - Hide canvas

### URL Structure
- Base: `http://<host>:18793/__openclaw__/canvas/`
- Files: `/__openclaw__/canvas/index.html` → `~/clawd/canvas/index.html`
- Nested: `/__openclaw__/canvas/games/snake.html` → `~/clawd/canvas/games/snake.html`

## Business Logic

### Distributed Architecture
1. **Canvas Host Server** (Port 18793): Serves static HTML/CSS/JS files
2. **Node Bridge** (Port 18790): Communicates canvas commands to nodes
3. **Node Apps**: Render content in WebView on target devices

### Network Binding Strategy
1. Determine bind mode from `gateway.bind` configuration
2. Choose appropriate interface: loopback → LAN → Tailscale
3. Construct URLs matching node network expectations
4. Handle hostname resolution for cross-device access

### Content Management
1. Serve static files from configurable root directory
2. Apply live reload watching for development workflows
3. Inject WebSocket client for automatic content updates
4. Handle file system changes with instant propagation

### Node Targeting
1. Discover available nodes with canvas capability
2. Route commands to specific node instances
3. Validate node connectivity before operations
4. Handle multi-node coordination for synchronized displays

## Data Structures

### Canvas Configuration
```typescript
interface CanvasConfig {
  enabled: boolean;            // canvas host service enabled
  port: number;                // HTTP server port (default 18793)
  root: string;                // content root directory path
  liveReload: boolean;         // automatic content refresh
  allowedOrigins?: string[];   // CORS configuration
}
```

### Canvas Command
```typescript
interface CanvasCommand {
  action: 'present' | 'hide' | 'navigate' | 'eval' | 'snapshot';
  nodeId: string;              // target node identifier
  target?: string;             // URL for present action
  url?: string;                // URL for navigate action
  javaScript?: string;         // code for eval action
  timeoutMs?: number;          // operation timeout
}
```

### Node Canvas State
```typescript
interface NodeCanvasState {
  nodeId: string;              // node identifier
  isVisible: boolean;          // canvas visibility status
  currentUrl?: string;         // currently displayed URL
  lastSnapshot?: string;       // base64 screenshot data
  capabilities: string[];      // supported canvas features
  webViewReady: boolean;       // WebView initialization status
}
```

### Live Reload Context
```typescript
interface LiveReloadContext {
  enabled: boolean;
  watchedPaths: string[];      // monitored directories
  connectedClients: string[];  // WebSocket client IDs
  wsServer: WebSocketServer;   // live reload server instance
}
```

## Constraints

### Network Dependencies
- **Multi-Host Access**: Nodes must reach canvas host server
- **Port Availability**: Port 18793 must be accessible across network
- **Hostname Resolution**: Proper DNS/Tailscale hostname configuration
- **Firewall Rules**: Network policies allowing cross-device HTTP access

### Platform Limitations
- **WebView Support**: Nodes must support modern WebView rendering
- **JavaScript Execution**: eval operations require script execution permissions
- **Resource Constraints**: Mobile devices have memory/processing limits
- **Screen Sizes**: Content must adapt to various display dimensions

### Content Restrictions
- **File System Access**: Canvas root directory must be accessible
- **CORS Policies**: Cross-origin requests limited by browser security
- **Protocol Limitations**: HTTPS content may have mixed content issues
- **JavaScript Sandbox**: Limited system access from canvas JavaScript

## Edge Cases

### Network Connectivity Issues
- **Node Offline**: Target nodes disconnected during canvas operations
- **Bridge Unavailable**: Node bridge service not responding
- **Host Unreachable**: Canvas host server inaccessible from nodes
- **Hostname Mismatch**: URL hostname doesn't match network binding
- **Port Conflicts**: Canvas host port already in use by other service

### Content Loading Problems
- **File Not Found**: Requested HTML files missing from canvas root
- **Permission Denied**: Insufficient file system permissions for content
- **Large Files**: Oversized content causing loading timeouts
- **Malformed HTML**: Invalid markup breaking WebView rendering
- **Resource Dependencies**: External resources failing to load

### WebView and JavaScript Issues
- **Script Execution Failures**: JavaScript errors preventing proper rendering
- **Memory Exhaustion**: Complex visualizations consuming excessive memory
- **WebView Crashes**: Native WebView component failing on nodes
- **Security Violations**: JavaScript attempting prohibited operations
- **Event Handler Conflicts**: Multiple scripts interfering with each other

### Live Reload Complications
- **File Watcher Failures**: File system monitoring breaking on some platforms
- **WebSocket Connection Loss**: Live reload clients disconnecting
- **Rapid File Changes**: Multiple file modifications causing reload conflicts
- **Directory Permission Issues**: Unable to monitor canvas root directory

## Technical Debt

### Architecture Complexity
- Multi-component system (host, bridge, nodes) creates debugging challenges
- Network binding logic complex and error-prone
- No centralized error handling across distributed components
- Limited monitoring of component health and connectivity

### Configuration Management
- Manual hostname resolution for network binding
- No automatic discovery of optimal network interfaces
- Configuration spread across multiple files and components
- Limited validation of canvas configuration settings

### Development Experience
- Live reload implementation basic and unreliable
- No integrated debugging tools for canvas content
- Limited error reporting for failed canvas operations
- No development server with enhanced debugging features

### Performance Limitations
- No content caching or optimization for mobile devices
- Sequential file serving without compression or bundling
- Memory leaks in long-running canvas sessions
- No resource usage monitoring or limiting

## Dependencies

### Core Dependencies
- **Canvas Host Server**: HTTP server for content delivery (Port 18793)
- **Node Bridge**: Communication layer to nodes (Port 18790)
- **File System**: Access to canvas root directory for content
- **WebView Components**: Native WebView rendering on target nodes

### Network Dependencies
- **HTTP Protocol**: For content delivery to nodes
- **WebSocket**: Live reload communication channel
- **Tailscale/VPN**: Cross-device network connectivity (optional)
- **DNS Resolution**: Hostname lookup for cross-device access

### System Dependencies
- **File Watcher**: chokidar for live reload file monitoring
- **JSON Processing**: Configuration parsing and command serialization
- **Image Processing**: Screenshot capture and encoding
- **Process Management**: Service lifecycle for canvas host

### Node Dependencies
- **Node Applications**: Mac/iOS/Android OpenClaw apps
- **WebView APIs**: Platform-specific web rendering components
- **Network Clients**: HTTP and WebSocket client implementations
- **UI Framework**: Native UI integration for canvas display

### Development Dependencies
- **Static File Server**: For content delivery during development
- **CORS Handling**: Cross-origin resource sharing support
- **Asset Pipeline**: CSS/JS processing and optimization (optional)
- **Debug Tools**: Browser developer tools integration

### Optional Dependencies
- **HTTPS/TLS**: Secure content delivery for production deployments
- **Content Compression**: Gzip/Brotli for bandwidth optimization
- **Authentication**: Access control for canvas content
- **Analytics**: Usage tracking and performance monitoring