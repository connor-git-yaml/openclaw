# BluCLI Skill Specification

## Intent

Provide command-line interface for controlling Bluesound/NAD BluOS audio players including playback control, volume management, device grouping, and streaming service integration. Enables home audio automation and control for multi-room audio systems.

## Interface

### Core Operations
- **Device Management**: Discover, list, and target BluOS players on network
- **Playback Control**: Play, pause, stop, skip tracks on selected devices
- **Volume Control**: Set, adjust, and mute volume levels
- **Device Grouping**: Create, modify, and dissolve multi-room audio groups
- **Service Integration**: Control TuneIn radio and other streaming services

### Command Structure
- `blu devices` - Discover and list available BluOS players
- `blu --device ID status` - Get current playback status
- `blu play|pause|stop` - Basic playback control
- `blu volume set LEVEL` - Set volume (0-100)
- `blu group status|add|remove` - Multi-room grouping operations
- `blu tunein search|play QUERY` - TuneIn radio integration

### Target Selection Priority
1. `--device <id|name|alias>` command flag
2. `BLU_DEVICE` environment variable
3. Default device from configuration file

### Output Modes
- Human-readable default formatting
- JSON output (`--json`) for programmatic access
- Structured data for automation workflows

## Business Logic

### Device Discovery and Management
1. Scan local network for BluOS-compatible devices
2. Maintain device registry with IDs, names, and capabilities
3. Handle device aliases and friendly names for user convenience
4. Validate device availability before operations

### Playback State Management
1. Query current playback status (playing, paused, stopped)
2. Retrieve track information (title, artist, album, duration)
3. Control playback state changes with error handling
4. Manage playlist and queue operations

### Volume Control Logic
1. Set absolute volume levels (0-100 scale)
2. Support relative volume adjustments (up/down)
3. Handle mute/unmute toggle operations
4. Respect volume limits and device capabilities

### Multi-Room Grouping
1. Create audio synchronization groups across multiple players
2. Designate master/slave relationships in groups
3. Propagate playback control to entire group
4. Handle group dissolution and member management

## Data Structures

### BluOS Device
```typescript
interface BluOSDevice {
  id: string;                  // unique device identifier
  name: string;                // device display name
  alias?: string;              // user-defined alias
  ipAddress: string;           // network IP address
  model: string;               // device model information
  version: string;             // firmware version
  capabilities: string[];      // supported feature list
  isGroupMaster: boolean;      // group leadership status
  groupMembers?: string[];     // IDs of grouped devices
}
```

### Playback Status
```typescript
interface PlaybackStatus {
  state: 'playing' | 'paused' | 'stopped';
  track?: {
    title: string;
    artist: string;
    album: string;
    duration: number;          // seconds
    position: number;          // current position in seconds
  };
  volume: number;              // current volume (0-100)
  isMuted: boolean;
  source: string;              // current input source
  service?: string;            // streaming service name
}
```

### Device Group
```typescript
interface AudioGroup {
  id: string;                  // group identifier
  name?: string;               // group display name
  master: string;              // master device ID
  members: string[];           // all device IDs in group
  syncStatus: 'synchronized' | 'syncing' | 'error';
  volumeSync: boolean;         // whether volumes sync across group
}
```

### Service Query
```typescript
interface ServiceQuery {
  service: 'tunein' | 'spotify' | 'tidal';
  query: string;               // search terms
  type?: 'station' | 'track' | 'album';
  results: ServiceResult[];
}

interface ServiceResult {
  id: string;
  title: string;
  type: string;
  url?: string;
  metadata?: Record<string, any>;
}
```

## Constraints

### Network Dependencies
- **Local Network**: Devices must be on same network segment as control client
- **UPnP/SSDP**: Requires multicast support for device discovery
- **HTTP API**: BluOS players expose REST API on specific ports
- **Real-time Updates**: Some features require persistent connections

### Device Limitations
- **Firmware Compatibility**: Features depend on BluOS firmware versions
- **Concurrent Control**: Multiple controllers may conflict
- **Device Capabilities**: Not all devices support all features
- **Power State**: Devices must be powered on and network-connected

### Service Integration
- **Authentication**: Streaming services may require separate login
- **Geographic Restrictions**: Content availability varies by region
- **API Limits**: Third-party services may impose usage limits
- **Content Rights**: DRM-protected content may have playback restrictions

## Edge Cases

### Network Connectivity Issues
- **Device Offline**: Players temporarily unavailable on network
- **Network Segmentation**: Devices on different VLANs/subnets
- **IP Address Changes**: DHCP reassigning device addresses
- **Firewall Blocking**: Network security blocking BluOS communication ports
- **WiFi Interference**: Wireless connectivity issues affecting control

### Grouping Complications
- **Group Sync Failures**: Audio synchronization problems between devices
- **Master Device Offline**: Group master becomes unavailable
- **Mixed Capabilities**: Grouping devices with different feature sets
- **Volume Conflicts**: Different volume levels when creating groups
- **Circular Groups**: Attempting to create invalid group relationships

### Service Integration Problems
- **Authentication Expiry**: Streaming service tokens becoming invalid
- **Service Unavailable**: Third-party services experiencing downtime
- **Content Not Found**: Search queries returning no results
- **Playback Rights**: Attempting to play unavailable content
- **Regional Blocking**: Content restricted in user's geographic location

### Command Execution Failures
- **Device Not Responding**: Target device unresponsive to commands
- **Invalid Parameters**: Volume levels outside acceptable range
- **State Conflicts**: Commands invalid for current playback state
- **Timeout Issues**: Long response times for device operations

## Technical Debt

### Discovery Reliability
- Device discovery may miss players under network stress
- No persistent device database between CLI sessions
- Limited retry logic for failed discovery attempts
- Inconsistent handling of device name changes

### Error Handling Deficiencies
- Poor error messages for network connectivity issues
- No graceful degradation when partial operations fail
- Limited context in error reporting for grouping failures
- Insufficient validation of user input parameters

### Configuration Management
- Basic configuration system with limited flexibility
- No user-friendly configuration setup wizard
- Environment variable naming could be more intuitive
- Missing configuration validation on startup

### Feature Coverage Gaps
- Limited streaming service integration beyond TuneIn
- No playlist management functionality
- Missing advanced equalizer/DSP controls
- No support for device firmware updates

## Dependencies

### Core Dependencies
- **blu CLI Binary**: Primary interface to BluOS ecosystem
- **Go Runtime**: Required for blu binary execution
- **Network Stack**: TCP/IP and UDP for device communication

### Network Discovery Dependencies
- **UPnP/SSDP**: Universal Plug and Play protocol for device discovery
- **Multicast Support**: Network infrastructure supporting multicast packets
- **HTTP Client**: For REST API communication with BluOS devices

### System Dependencies
- **Operating System**: Cross-platform support (Windows, macOS, Linux)
- **Network Permissions**: Ability to send/receive network packets
- **DNS Resolution**: For resolving device hostnames

### External Service Dependencies
- **TuneIn**: Internet radio service integration
- **Streaming Services**: Optional integration with Spotify, Tidal, etc.
- **Internet Connectivity**: For cloud-based services and content

### Development Dependencies
- **Go Toolchain**: For building blu from source
- **Git**: For source code management
- **Network Testing Tools**: For development and debugging
- **BluOS Documentation**: API reference for development

### Optional Dependencies
- **Home Automation Systems**: Integration with smart home platforms
- **Audio Processing**: Advanced audio format support
- **Configuration Management**: External configuration systems
- **Logging Framework**: Enhanced logging capabilities