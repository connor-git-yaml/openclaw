# Camsnap Skill Specification

## Intent

Provide command-line interface for capturing images and video clips from RTSP/ONVIF IP cameras with motion detection capabilities. Enables security monitoring, automation triggers, and media capture workflows for surveillance and home automation systems.

## Interface

### Core Operations
- **Camera Management**: Add, configure, and discover network cameras
- **Image Capture**: Take snapshots from configured cameras
- **Video Recording**: Create time-based or event-triggered video clips
- **Motion Detection**: Monitor cameras for movement with configurable sensitivity
- **Health Monitoring**: Check camera connectivity and stream health

### Command Structure
- `camsnap add --name NAME --host IP --user USER --pass PASS` - Add camera configuration
- `camsnap discover --info` - Discover ONVIF cameras on network
- `camsnap snap CAMERA --out FILE.jpg` - Capture single frame
- `camsnap clip CAMERA --dur 5s --out FILE.mp4` - Record video clip
- `camsnap watch CAMERA --threshold 0.2 --action COMMAND` - Motion monitoring
- `camsnap doctor --probe` - Diagnose camera connectivity issues

### Configuration Management
- Configuration file: `~/.config/camsnap/config.yaml`
- Per-camera settings for authentication, resolution, encoding
- Motion detection thresholds and action triggers

## Business Logic

### Camera Discovery and Configuration
1. Scan local network for ONVIF-compatible cameras
2. Probe camera capabilities (resolutions, formats, features)
3. Store camera configurations with authentication credentials
4. Validate camera connectivity during configuration

### Media Capture Workflow
1. Establish RTSP connection to target camera
2. Extract single frames or continuous video streams
3. Apply format conversion and encoding via FFmpeg
4. Save media files to specified output paths
5. Handle capture failures and retry logic

### Motion Detection System
1. Continuously monitor camera feed for frame changes
2. Calculate motion intensity using configurable algorithms
3. Trigger actions when motion threshold exceeded
4. Implement cooldown periods to prevent excessive triggering
5. Log motion events with timestamps and intensity scores

### Health Monitoring
1. Perform connectivity checks on configured cameras
2. Test RTSP stream accessibility and quality
3. Monitor frame rates and stream stability
4. Report camera status and diagnostic information

## Data Structures

### Camera Configuration
```typescript
interface CameraConfig {
  name: string;                // user-friendly camera name
  host: string;                // IP address or hostname
  port?: number;               // RTSP port (default 554)
  username: string;            // authentication username
  password: string;            // authentication password
  rtspPath?: string;           // custom RTSP path
  resolution?: string;         // preferred capture resolution
  fps?: number;                // frames per second
  encoding?: string;           // video encoding preference
}
```

### Capture Parameters
```typescript
interface CaptureParams {
  camera: string;              // target camera name
  outputPath: string;          // destination file path
  duration?: string;           // clip duration (e.g., "5s", "1m")
  quality?: number;            // image/video quality (1-100)
  format?: string;             // output format (jpg, mp4, etc.)
  timestamp?: boolean;         // embed timestamp overlay
}
```

### Motion Event
```typescript
interface MotionEvent {
  camera: string;              // source camera name
  timestamp: Date;             // event detection time
  intensity: number;           // motion intensity score (0-1)
  duration: number;            // motion duration in milliseconds
  triggerAction?: string;      // executed action command
  previewPath?: string;        // captured frame during motion
}
```

### Camera Status
```typescript
interface CameraStatus {
  name: string;                // camera identifier
  online: boolean;             // connectivity status
  streamActive: boolean;       // RTSP stream availability
  lastSeen: Date;              // last successful connection
  errorMessage?: string;       // diagnostic error information
  frameRate?: number;          // current FPS
  resolution?: string;         // active resolution
}
```

## Constraints

### Network Dependencies
- **IP Camera Access**: Requires network connectivity to camera devices
- **RTSP Protocol**: Cameras must support RTSP streaming
- **Authentication**: Valid camera credentials required
- **Bandwidth**: Sufficient network bandwidth for video streams

### Software Dependencies
- **FFmpeg**: Required for video processing and format conversion
- **ONVIF Support**: Cameras should support ONVIF for discovery features
- **Codec Support**: Video codecs must be compatible with FFmpeg
- **File System**: Write access to output directories

### Hardware Limitations
- **Processing Power**: Video encoding/decoding requires CPU resources
- **Storage Space**: Video clips consume significant disk space
- **Memory Usage**: Continuous monitoring uses memory for frame buffers
- **Concurrent Streams**: Limited simultaneous camera connections

## Edge Cases

### Network Connectivity Issues
- **Camera Offline**: Target cameras powered off or disconnected
- **Network Segmentation**: Cameras on different network subnets
- **DHCP Changes**: Camera IP addresses changing dynamically
- **Bandwidth Saturation**: Network congestion affecting stream quality
- **Authentication Failures**: Invalid or expired camera credentials

### Stream Processing Problems
- **Codec Incompatibility**: Unsupported video formats from cameras
- **Resolution Mismatches**: Requested resolution not available
- **Frame Rate Issues**: Inconsistent or low frame rates
- **Stream Corruption**: Damaged or incomplete video data
- **Timeout Errors**: Long delays in stream establishment

### Storage and Performance
- **Disk Full**: Insufficient storage space for video capture
- **File Permission Errors**: Unable to write to output directories
- **Concurrent Access**: Multiple processes accessing same camera
- **Resource Exhaustion**: High CPU/memory usage during processing
- **Long-running Operations**: Motion watch processes consuming resources

### Motion Detection Edge Cases
- **False Positives**: Environmental changes triggering false motion
- **Sensitivity Issues**: Motion threshold too high/low for conditions
- **Action Command Failures**: Triggered actions failing to execute
- **Rapid Triggering**: Multiple motion events in quick succession
- **Lighting Changes**: Day/night transitions affecting detection

## Technical Debt

### Configuration Management
- Plain text password storage in configuration files
- No configuration validation on startup
- Limited configuration migration tools between versions
- Manual network discovery without persistent device tracking

### Error Handling and Recovery
- Generic error messages for diverse failure modes
- No automatic retry mechanisms for transient failures
- Limited logging for debugging connectivity issues
- Poor recovery from partial stream failures

### Performance Optimization
- No stream caching or buffering optimizations
- Sequential processing instead of parallel camera handling
- Memory leaks in long-running motion detection processes
- Inefficient motion detection algorithms for high-resolution streams

### Feature Completeness
- Limited audio capture support
- No advanced motion detection zones or masking
- Missing stream quality adaptation based on network conditions
- No integration with popular home automation platforms

## Dependencies

### Core Dependencies
- **camsnap Binary**: Primary CLI tool for camera operations
- **FFmpeg**: Video processing and format conversion engine
- **RTSP Client Libraries**: For camera stream connectivity

### Network Dependencies
- **ONVIF Protocol**: For camera discovery and capability detection
- **RTSP Protocol**: For video stream access
- **HTTP/HTTPS**: For camera API communication
- **Network Discovery**: For automatic camera detection

### System Dependencies
- **File System**: For configuration storage and media output
- **Process Management**: For background motion monitoring
- **Network Stack**: TCP/UDP connectivity to cameras
- **Codec Libraries**: Video/audio encoding and decoding

### Configuration Dependencies
- **YAML Parser**: For configuration file processing
- **Credential Storage**: Secure storage of camera passwords
- **Path Resolution**: File system path management
- **Environment Variables**: Runtime configuration override

### Optional Dependencies
- **Notification System**: For motion event alerts
- **Cloud Storage**: For backup of captured media
- **Database**: For motion event logging and history
- **Home Automation**: Integration with smart home platforms
- **Image Processing**: Advanced motion detection algorithms

### Development Dependencies
- **Go Toolchain**: For building camsnap from source
- **Testing Framework**: For camera simulation and testing
- **Documentation Tools**: For API and configuration documentation