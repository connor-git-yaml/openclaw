# SonosCLI Skill Specification

## Intent

Provide comprehensive Sonos speaker control through the sonoscli tool for network device discovery, playback control, volume management, and multi-room audio grouping in smart home environments.

## Interface

### Core Operations
- **Device Discovery**: Find and list Sonos speakers on network
- **Playback Control**: Play, pause, skip, and stop audio content
- **Volume Management**: Adjust volume levels for individual speakers or groups
- **Group Management**: Create, modify, and dissolve speaker groups
- **Status Monitoring**: Check current playback state and device information

### CLI Structure
- **Discovery**: `sonos discover` - Find available Sonos devices
- **Status**: `sonos status --room "Living Room"` - Check device state
- **Playback**: `sonos play|pause|next|previous --room "Kitchen"`
- **Volume**: `sonos volume 50 --room "Bedroom"`
- **Grouping**: `sonos group "Living Room" "Kitchen" "Dining Room"`

## Business Logic

### Network Discovery and Device Management
1. Scan local network for Sonos devices using UPnP/SSDP protocols
2. Identify speaker capabilities, zones, and current configurations
3. Maintain device state cache and handle device availability changes
4. Process device naming and room assignment updates

### Audio Playback and Content Control
1. Send playback commands to individual speakers or speaker groups
2. Handle queue management and playlist operations
3. Process streaming service integration and content selection
4. Manage crossfade, repeat, and shuffle settings

### Volume and Audio Configuration
1. Control individual speaker volume levels and muting
2. Manage group volume with synchronized level changes
3. Handle volume ramping and fade operations
4. Process equalizer and audio enhancement settings

### Multi-Room Audio and Grouping
1. Create and manage speaker groups for synchronized playback
2. Handle group coordinator selection and management
3. Process group volume and playback synchronization
4. Manage zone bridging and audio routing

## Data Structures

### Sonos Device
```typescript
interface SonosDevice {
  uuid: string; // device unique identifier
  name: string; // user-assigned room name
  ip: string; // device IP address
  model: string; // speaker model
  software: string; // firmware version
  capabilities: DeviceCapabilities; // supported features
  state: DeviceState; // current device state
  group?: string; // group membership
}

interface DeviceCapabilities {
  playback: boolean; // audio playback support
  lineIn: boolean; // line input availability
  headphone: boolean; // headphone output
  microphone: boolean; // microphone input
  battery: boolean; // battery powered
}
```

### Playback State
```typescript
interface DeviceState {
  transport: 'playing' | 'paused' | 'stopped'; // playback state
  volume: number; // volume level (0-100)
  muted: boolean; // mute state
  crossfade: boolean; // crossfade enabled
  repeat: 'none' | 'one' | 'all'; // repeat mode
  shuffle: boolean; // shuffle enabled
  currentTrack?: TrackInfo; // playing track
  queue: QueueInfo; // playback queue
}

interface TrackInfo {
  title: string; // track title
  artist: string; // track artist
  album: string; // album name
  duration: number; // track length (seconds)
  position: number; // playback position (seconds)
  uri: string; // content URI
  albumArt?: string; // album artwork URL
}
```

### Group Configuration
```typescript
interface SpeakerGroup {
  id: string; // group identifier
  coordinator: string; // group coordinator UUID
  members: string[]; // member device UUIDs
  name: string; // group display name
  volume: number; // group volume level
  state: GroupState; // group playback state
}

interface GroupState {
  transport: 'playing' | 'paused' | 'stopped';
  synchronized: boolean; // synchronization status
  currentSource: string; // active content source
  queue: QueueInfo; // shared queue
}
```

### Queue Information
```typescript
interface QueueInfo {
  size: number; // queue length
  position: number; // current position
  tracks: TrackInfo[]; // queued tracks
  canSkip: boolean; // skip capability
  canSeek: boolean; // seek capability
}
```

## Constraints

### Network and Connectivity Requirements
- **Local Network**: Sonos devices must be on same LAN segment
- **UPnP/SSDP**: Requires multicast networking support
- **Firewall**: Network firewall may block device discovery
- **Wireless Stability**: Wi-Fi connectivity affects device responsiveness

### Device and Firmware Limitations
- **Model Differences**: Older Sonos devices have limited capabilities
- **Firmware Compatibility**: Commands may vary across software versions
- **Group Limits**: Maximum number of speakers per group (32 devices)
- **Zone Configuration**: Device zone assignments affect grouping options

### Content and Service Restrictions
- **Streaming Services**: Limited to configured/authenticated services
- **Local Content**: File format and codec support variations
- **DRM Content**: Protected content playback limitations
- **Queue Limits**: Maximum queue size restrictions

## Edge Cases

### Network Discovery Issues
- **No Devices Found**: Network isolation or device offline
- **Partial Discovery**: Some devices not detected due to network issues
- **IP Changes**: Device IP addresses changing affecting cached connections
- **Network Segmentation**: Devices on different network segments
- **Firewall Blocking**: Security software preventing device communication

### Device State Inconsistencies
- **Command Failures**: Devices not responding to control commands
- **State Desync**: Device state not reflecting actual playback status
- **Group Coordination**: Group member synchronization problems
- **Volume Inconsistency**: Individual vs group volume conflicts
- **Transport Errors**: Playback state transitions failing

### Content and Playback Problems
- **Content Unavailable**: Streaming service or local content inaccessible
- **Format Unsupported**: Audio format not supported by device
- **Queue Management**: Queue operations failing or corrupting playlist
- **Authentication**: Streaming service login issues
- **Buffering Issues**: Network bandwidth problems affecting playback

## Technical Debt

### Device Management Complexity
- Manual device discovery without persistent device registry
- No automatic device state synchronization or monitoring
- Limited error handling for device communication failures
- Basic retry mechanisms for network operations

### Command Interface Limitations
- Simple command structure without advanced automation features
- No batch operations for multiple device management
- Limited scriptability and integration capabilities
- Manual room/device name resolution

### State Management Issues
- No persistent state tracking across CLI sessions
- Limited real-time event handling for device changes
- Basic caching without intelligent cache invalidation
- Manual state refresh requirements

## Dependencies

### Core Dependencies
- **sonoscli (sonos)**: Sonos CLI control tool
- **Network Connectivity**: Local network access for device communication
- **UPnP/SSDP Support**: Network protocol support for device discovery
- **HTTP Client**: Web requests for device control APIs

### System Dependencies
- **Operating System**: Cross-platform networking stack compatibility
- **Multicast Support**: Network multicast routing for device discovery
- **DNS Resolution**: Local network name resolution capabilities
- **Port Access**: Network port availability for device communication

### Hardware Dependencies
- **Sonos Devices**: Compatible Sonos speakers and components
- **Network Infrastructure**: Wi-Fi or wired network connectivity
- **Router Configuration**: UPnP/multicast support in network equipment
- **Bandwidth**: Sufficient network bandwidth for audio streaming

### Optional Dependencies
- **Home Automation**: Integration with smart home systems
- **Voice Control**: Voice assistant integration for hands-free control
- **Mobile Apps**: Sonos mobile app for advanced configuration
- **Streaming Services**: Music service subscriptions and authentication