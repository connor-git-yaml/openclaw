# Peekaboo Skill Specification

## Intent

Provide comprehensive macOS UI automation and screen capture capabilities through the Peekaboo CLI tool. Enables programmatic control of applications, windows, menus, and user interface elements with computer vision, gesture automation, and intelligent element targeting for testing and automation workflows.

## Interface

### Core Capabilities
- **UI Inspection**: Visual element identification and annotation with snapshot analysis
- **Mouse Automation**: Clicking, dragging, scrolling, and gesture automation
- **Keyboard Control**: Text input, hotkey combinations, and special key sequences
- **Window Management**: Application launching, window positioning, and workspace control
- **Screen Capture**: Screenshots, live capture, and video frame extraction

### Command Categories
- **Core**: bridge, capture, clean, config, image, learn, list, permissions, run, sleep, tools
- **Interaction**: click, drag, hotkey, move, paste, press, scroll, swipe, type
- **System**: app, clipboard, dialog, dock, menu, menubar, open, space, visualizer, window
- **Vision**: see (annotated UI maps and analysis)

### Targeting System
- **Element Targeting**: `--on ID` for visual element identification
- **Coordinate Targeting**: `--coords x,y` for precise positioning
- **Application Context**: `--app NAME`, `--pid`, `--window-title` for app-specific operations
- **Snapshot Reference**: `--snapshot ID` for cached visual context

### Permission Requirements
- **Screen Recording**: Required for screen capture and visual analysis
- **Accessibility**: Required for UI automation and element interaction

## Business Logic

### Visual Element Recognition and Targeting
1. Capture and analyze UI screenshots with computer vision
2. Identify clickable elements, buttons, and interactive components
3. Generate annotated snapshots with element IDs for targeting
4. Cache visual snapshots for efficient repeated operations
5. Handle dynamic UI changes and element state variations

### Automated Input and Interaction
1. Execute precise mouse movements with human-like motion profiles
2. Perform keyboard input with natural typing patterns and delays
3. Handle complex gesture sequences and multi-touch interactions
4. Coordinate timing and synchronization across multiple input events
5. Implement error recovery and retry logic for failed interactions

### Application and Window Management
1. Launch and manage macOS applications with proper focus handling
2. Control window positioning, sizing, and workspace distribution
3. Navigate application menus and system interface elements
4. Handle application state changes and lifecycle management
5. Coordinate multiple application interactions and context switching

### Screen Capture and Analysis
1. Capture high-quality screenshots with retina display support
2. Perform live capture with motion detection and frame optimization
3. Extract and analyze video frames from recorded content
4. Apply AI-powered visual analysis and content understanding
5. Generate visual annotations and element identification overlays

### Permission and Security Management
1. Validate and request necessary macOS permission grants
2. Handle sandboxing and security restrictions gracefully
3. Provide clear guidance for permission configuration
4. Implement secure credential storage for AI analysis services
5. Respect user privacy and data protection requirements

## Data Structures

### UI Element
```typescript
interface UIElement {
  id: string;                  // unique element identifier
  type: 'button' | 'text' | 'image' | 'menu' | 'field' | 'other';
  bounds: Rectangle;           // element bounding box
  center: Point;               // clickable center point
  isClickable: boolean;        // interaction capability
  isVisible: boolean;          // visibility status
  text?: string;               // visible text content
  accessibility?: AccessibilityInfo;
  confidence: number;          // detection confidence (0-1)
}

interface Rectangle {
  x: number;                   // left coordinate
  y: number;                   // top coordinate
  width: number;               // element width
  height: number;              // element height
}

interface Point {
  x: number;                   // horizontal coordinate
  y: number;                   // vertical coordinate
}
```

### Window Information
```typescript
interface WindowInfo {
  id: number;                  // window identifier
  title: string;               // window title
  appName: string;             // owning application
  pid: number;                 // process identifier
  bounds: Rectangle;           // window bounds
  isVisible: boolean;          // visibility status
  isMinimized: boolean;        // minimization state
  isFocused: boolean;          // focus status
  spaceIndex?: number;         // assigned workspace
  level: number;               // window layer level
}
```

### Capture Configuration
```typescript
interface CaptureConfig {
  mode: 'screen' | 'window' | 'frontmost' | 'region';
  screenIndex?: number;        // target display
  windowId?: number;           // target window
  region?: Rectangle;          // capture region
  format: 'png' | 'jpg';       // output format
  retina: boolean;             // high-resolution capture
  captureEngine: 'auto' | 'classic' | 'cg' | 'modern' | 'sckit';
  outputPath?: string;         // file destination
  analyze?: string;            // AI analysis prompt
  annotate: boolean;           // element annotation
}
```

### Motion Profile
```typescript
interface MotionProfile {
  type: 'human' | 'linear';     // movement style
  duration: number;            // movement duration (ms)
  steps?: number;              // interpolation steps
  easing?: 'ease-in' | 'ease-out' | 'ease-in-out';
  jitter?: number;             // random variation amount
  acceleration: number;        // motion acceleration
  deceleration: number;        // motion deceleration
}
```

### Automation Command
```typescript
interface AutomationCommand {
  type: 'click' | 'type' | 'drag' | 'scroll' | 'hotkey';
  target?: ElementTarget;      // target specification
  parameters: CommandParameters;
  timing: TimingConfig;        // execution timing
  retry: RetryConfig;          // failure handling
  validation?: ValidationRule; // success criteria
}

interface ElementTarget {
  method: 'id' | 'coords' | 'query';
  value: string | Point;       // target value
  appContext?: string;         // application context
  windowContext?: number;      // window context
}

interface CommandParameters {
  text?: string;               // text input
  keys?: string[];             // key combinations
  direction?: 'up' | 'down' | 'left' | 'right';
  amount?: number;             // scroll amount
  from?: ElementTarget;        // drag source
  to?: ElementTarget;          // drag destination
}
```

### Snapshot Analysis
```typescript
interface SnapshotAnalysis {
  snapshotId: string;          // cached snapshot identifier
  timestamp: Date;             // capture timestamp
  elements: UIElement[];       // detected elements
  annotations: Annotation[];   // visual annotations
  aiAnalysis?: string;         // AI-generated analysis
  metadata: SnapshotMetadata;  // capture metadata
}

interface Annotation {
  id: string;                  // annotation identifier
  type: 'element' | 'highlight' | 'text';
  bounds: Rectangle;           // annotation area
  label: string;               // annotation text
  color: string;               // display color
  fontSize?: number;           // text size
}
```

## Constraints

### macOS Platform Dependencies
- **Operating System**: Requires macOS for Accessibility and Screen Recording APIs
- **Permission Requirements**: Screen Recording and Accessibility permissions mandatory
- **API Limitations**: Dependent on macOS UI automation framework capabilities
- **Application Compatibility**: Some applications may have automation restrictions

### Visual Recognition Limitations
- **UI Complexity**: Complex interfaces may confuse element detection
- **Dynamic Content**: Rapidly changing UIs difficult to target reliably
- **Resolution Dependencies**: Element detection accuracy varies with screen resolution
- **Color and Contrast**: Visual elements with poor contrast may be missed

### Performance and Resource Constraints
- **CPU Intensive**: Computer vision and image processing require significant CPU
- **Memory Usage**: Screen captures and analysis consume substantial RAM
- **Disk Space**: Snapshot caching and video capture require storage space
- **Real-time Requirements**: Live automation requires responsive execution

### Security and Privacy Limitations
- **System Permissions**: Extensive permissions required for full functionality
- **Application Sandboxing**: Some applications prevent external automation
- **User Privacy**: Screen recording capabilities raise privacy concerns
- **Credential Access**: AI analysis may require external service credentials

## Edge Cases

### Permission and Access Issues
- **Permission Denied**: Screen Recording or Accessibility permissions not granted
- **Permission Revoked**: Previously granted permissions removed by user
- **Application Blocking**: Target applications actively preventing automation
- **System Updates**: macOS updates changing permission requirements
- **Privacy Settings**: System privacy settings blocking UI access

### Visual Recognition Problems
- **Element Misidentification**: Computer vision incorrectly identifying UI elements
- **Overlapping Elements**: Multiple elements occupying same screen coordinates
- **Dynamic UI Changes**: Interface changes during automation execution
- **Scale and Resolution**: UI elements appearing different at various resolutions
- **Theme Changes**: Dark/light mode affecting element recognition

### Application State Complications
- **Application Crashes**: Target applications crashing during automation
- **Focus Loss**: Applications losing focus affecting automation reliability
- **Modal Dialogs**: Unexpected dialogs blocking automation progress
- **Context Switching**: Applications changing state unexpectedly
- **Resource Exhaustion**: Applications becoming unresponsive under automation load

### Timing and Synchronization Issues
- **Race Conditions**: UI changes occurring faster than automation can react
- **Animation Interference**: UI animations affecting element targeting accuracy
- **Network Latency**: Web applications with slow loading affecting automation
- **System Performance**: High CPU/memory usage affecting automation timing
- **Concurrent Operations**: Multiple automation processes interfering

## Technical Debt

### Computer Vision Accuracy
- Basic element detection without advanced machine learning optimization
- Limited handling of complex UI patterns and custom controls
- No adaptive learning from automation failures and corrections
- Basic confidence scoring without sophisticated reliability metrics

### Error Handling and Recovery
- Generic error messages without specific guidance for resolution
- Limited retry strategies for different types of automation failures
- No automatic adjustment of parameters based on environmental conditions
- Basic fallback mechanisms when primary automation methods fail

### Performance Optimization
- No intelligent caching strategies for frequently accessed UI elements
- Basic parallelization of capture and analysis operations
- Memory usage not optimized for long-running automation sessions
- Limited optimization for high-resolution and multi-monitor setups

### User Experience and Integration
- Command-line only interface requiring technical expertise
- Limited integration with popular automation and testing frameworks
- No graphical interface for visual automation workflow design
- Basic reporting and analytics for automation execution and success rates

## Dependencies

### Core Dependencies
- **Peekaboo Binary**: Primary CLI tool for macOS UI automation
- **macOS Accessibility Framework**: System APIs for UI automation
- **Core Graphics**: Screen capture and image processing APIs
- **Vision Framework**: Computer vision for element detection

### System Dependencies
- **macOS**: Operating system platform requirement (Darwin only)
- **Screen Recording Permission**: System permission for screen capture
- **Accessibility Permission**: System permission for UI automation
- **File System Access**: Snapshot caching and configuration storage

### Image Processing Dependencies
- **Core Image**: Image analysis and processing framework
- **ImageIO**: Image format reading and writing
- **Metal/OpenGL**: GPU acceleration for image processing
- **Video Toolbox**: Video capture and frame extraction

### Network and AI Dependencies
- **HTTP Client**: Communication with AI analysis services
- **JSON Processing**: Configuration and response data handling
- **Cryptographic Libraries**: Secure credential storage
- **WebSocket**: Real-time communication for live capture

### Optional Dependencies
- **Homebrew**: Package manager for installation (steipete/tap)
- **FFmpeg**: Advanced video processing capabilities
- **AI Analysis Services**: External computer vision and analysis APIs
- **Testing Frameworks**: Integration with automated testing tools

### Development Dependencies
- **Swift/Objective-C**: Implementation language and system frameworks
- **Build Tools**: Compilation and packaging for distribution
- **Testing Framework**: Automation reliability and accuracy testing
- **Documentation**: Usage guides and API reference materials