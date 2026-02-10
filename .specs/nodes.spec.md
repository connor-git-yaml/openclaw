# Nodes Module Specification

## Intent
The Nodes module manages distributed computing capabilities for OpenClaw, enabling remote device pairing, resource delegation, and cross-device coordination. It provides secure device discovery, pairing workflows, remote command execution, media capture, location services, and screen recording. The module enables OpenClaw agents to leverage capabilities across multiple devices in a trusted network.

## Interface

### Core Types
```typescript
export type NodeAction = 'status' | 'describe' | 'pending' | 'approve' | 'reject' | 
  'notify' | 'camera_snap' | 'camera_list' | 'camera_clip' | 'screen_record' | 
  'location_get' | 'run' | 'invoke';

export type NodeRequest = {
  action: NodeAction;
  node?: string;
  command?: string[];
  cwd?: string;
  env?: string[];
  timeoutMs?: number;
  commandTimeoutMs?: number;
  invokeCommand?: string;
  invokeParamsJson?: string;
  invokeTimeoutMs?: number;
  gatewayUrl?: string;
  gatewayToken?: string;
};

export type NodeInfo = {
  id: string;
  name: string;
  type: 'mobile' | 'desktop' | 'server';
  platform: string;
  capabilities: NodeCapability[];
  status: 'online' | 'offline' | 'pairing';
  lastSeen: Date;
  location?: GeoLocation;
};

export type NodeCapability = 'exec' | 'camera' | 'location' | 'screen' | 'notification' | 'storage';
```

### Key Functions
- `discoverNodes()` - Scan for available nodes on network
- `pairNode(nodeId, approvalToken)` - Initiate secure pairing process
- `executeRemote(nodeId, command, options)` - Run command on remote node
- `captureMedia(nodeId, type, options)` - Capture camera/screen from node
- `getNodeLocation(nodeId)` - Request location from mobile node
- `sendNotification(nodeId, message, options)` - Send notification to node
- `invokeNodeCapability(nodeId, capability, params)` - Execute node-specific function

### Node Status Operations
- `status` - Get overview of all paired nodes
- `describe` - Get detailed info about specific node
- `pending` - List nodes awaiting pairing approval
- `approve/reject` - Handle pairing requests

### Media Capture Operations
- `camera_snap` - Take photo from node camera
- `camera_list` - List available cameras on node
- `camera_clip` - Record video from node camera
- `screen_record` - Capture screen recording from node

## Business Logic

### Device Discovery and Pairing
1. **Network Scanning**: Discover OpenClaw-capable devices on local network
2. **Capability Detection**: Identify available services on discovered nodes
3. **Pairing Request**: Initiate secure pairing with authentication tokens
4. **Approval Workflow**: User approval required on both devices
5. **Key Exchange**: Establish secure communication channel
6. **Capability Registration**: Register available node capabilities

### Remote Execution Management
- **Command Delegation**: Route execution to appropriate node
- **Security Context**: Maintain security boundaries across nodes
- **Resource Monitoring**: Track resource usage on remote nodes
- **Failure Recovery**: Handle node disconnection during execution
- **Result Aggregation**: Collect and merge results from multiple nodes

### Media Capture Coordination
- **Camera Management**: Access front/back cameras on mobile devices
- **Screen Capture**: Record desktop screens and mobile displays
- **Quality Control**: Adaptive quality based on network conditions
- **Storage Management**: Efficient transfer and storage of media files
- **Privacy Controls**: User consent required for media capture

### Location Services Integration
- **GPS Coordination**: Request location data from mobile nodes
- **Accuracy Levels**: Configurable precision (coarse/balanced/precise)
- **Privacy Protection**: Location sharing with explicit user consent
- **Historical Tracking**: Optional location history for workflow automation

### Cross-Node Communication
- **Message Routing**: Efficient message delivery between nodes
- **State Synchronization**: Consistent state across distributed system
- **Network Optimization**: Adaptive protocols based on connection quality
- **Offline Handling**: Graceful degradation when nodes are unavailable

## Data Structures

### Node Registry
```typescript
type NodeRegistry = {
  nodes: Map<string, NodeInfo>;
  capabilities: Map<string, Set<NodeCapability>>;
  pairings: Map<string, PairingState>;
  sessions: Map<string, RemoteSession>;
};
```

### Pairing State
```typescript
type PairingState = {
  nodeId: string;
  requestTime: Date;
  approvalToken: string;
  status: 'pending' | 'approved' | 'rejected' | 'expired';
  requesterInfo: NodeInfo;
  approverInfo?: NodeInfo;
};
```

### Remote Session
```typescript
type RemoteSession = {
  id: string;
  nodeId: string;
  type: 'exec' | 'media' | 'location' | 'notification';
  startTime: Date;
  status: 'active' | 'completed' | 'failed';
  metadata: Record<string, any>;
};
```

### Media Capture Request
```typescript
type MediaCaptureRequest = {
  nodeId: string;
  deviceId?: string;
  facing?: 'front' | 'back' | 'both';
  quality?: number;
  maxWidth?: number;
  durationMs?: number;
  fps?: number;
  includeAudio?: boolean;
  outPath?: string;
};
```

## Constraints

### Network Requirements
- **Local Network**: Nodes must be on same trusted network
- **Firewall Configuration**: Specific ports must be open for discovery
- **Bandwidth Limits**: Media capture requires adequate bandwidth
- **Latency Tolerance**: Some operations sensitive to network delays

### Security Restrictions
- **Pairing Authentication**: Strong authentication required for new pairings
- **Capability Permissions**: Granular permissions for each node capability
- **Communication Encryption**: All inter-node communication encrypted
- **Access Control**: User consent required for sensitive operations

### Platform Limitations
- **iOS Restrictions**: Limited background execution and media access
- **Android Permissions**: Runtime permissions for camera and location
- **Desktop Security**: Platform-specific security model compliance
- **Network Discovery**: Platform-dependent network discovery mechanisms

### Resource Constraints
- **Battery Management**: Optimize for mobile device battery life
- **Storage Limits**: Efficient media storage and cleanup
- **CPU Usage**: Background processing limits on mobile devices
- **Memory Usage**: Bounded memory consumption for long-running operations

## Edge Cases

### Network Connectivity Issues
- **Intermittent Connection**: Handling of unstable network conditions
- **Node Disappearance**: Graceful handling when nodes go offline
- **Network Partition**: Recovery from network splits
- **Bandwidth Saturation**: Adaptive quality for media operations

### Pairing Workflow Failures
- **Timeout Scenarios**: Pairing requests that time out
- **Rejection Handling**: Clean up after pairing rejection
- **Duplicate Requests**: Handling multiple pairing attempts
- **Token Expiration**: Proper cleanup of expired pairing tokens

### Cross-Platform Compatibility
- **Version Mismatch**: Different OpenClaw versions on nodes
- **Capability Differences**: Features not available on all platforms
- **Protocol Evolution**: Handling protocol version differences
- **Format Incompatibility**: Media format support variations

### Security Incident Response
- **Compromise Detection**: Identifying potentially compromised nodes
- **Revocation Process**: Emergency revocation of node access
- **Audit Trail**: Maintaining logs for security investigations
- **Isolation Procedures**: Quarantining suspected malicious nodes

## Technical Debt

### Known Issues
- **Discovery Reliability**: Inconsistent node discovery across networks
- **iOS Background Limits**: Reduced functionality when iOS app backgrounded
- **Media Sync**: Large media files don't handle interruptions well
- **Error Reporting**: Inconsistent error messages across platforms

### Performance Optimizations Needed
- **Connection Pooling**: Reuse connections for multiple operations
- **Caching Strategy**: Cache node capabilities and status
- **Compression**: Better compression for media and data transfer
- **Batch Operations**: Group multiple requests for efficiency

### Architecture Improvements Required
- **Service Discovery**: More robust service discovery mechanism
- **Load Balancing**: Distribute work across available nodes
- **Fault Tolerance**: Better handling of partial node failures
- **State Management**: Improved distributed state consistency

### Security Enhancements Needed
- **Certificate Management**: Automated certificate rotation
- **Intrusion Detection**: Real-time monitoring for suspicious activity
- **Zero-Trust Model**: Implement zero-trust security principles
- **Audit Logging**: Comprehensive audit trail for all operations

## Dependencies

### Internal Dependencies
- **Gateway**: WebSocket communication and node registration
- **Security**: Authentication, authorization, and encryption
- **Config**: Node pairing configuration and capability settings
- **Agents**: Integration with agent execution context

### External Dependencies
- **Network Discovery**: Platform-specific service discovery (Bonjour/mDNS)
- **WebSocket**: Real-time communication with remote nodes
- **Encryption Libraries**: TLS/SSL for secure communication
- **Media Processing**: Camera and screen capture libraries

### Platform Dependencies
- **iOS**: Core Location, AVFoundation, Network framework
- **Android**: Location Services, Camera2 API, MediaProjection
- **macOS**: Core Location, AVFoundation, ScreenCaptureKit
- **Linux**: V4L2 for camera, X11/Wayland for screen capture