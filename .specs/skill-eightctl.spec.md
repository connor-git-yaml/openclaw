# Eightctl Skill Specification

## Intent

Provide command-line interface for controlling Eight Sleep smart mattress pods including temperature regulation, sleep tracking, alarm management, and scheduling. Enables sleep automation and health monitoring through programmatic access to Eight Sleep ecosystem.

## Interface

### Core Operations
- **Status Monitoring**: Get current pod status, temperature, and sleep metrics
- **Temperature Control**: Set heating/cooling levels for personalized comfort
- **Power Management**: Turn pod heating/cooling systems on/off
- **Alarm Management**: Create, list, and dismiss alarms
- **Schedule Control**: Manage temperature and sleep schedules

### Command Structure
- `eightctl status` - Display current pod status and metrics
- `eightctl on|off` - Control pod power state
- `eightctl temp DEGREES` - Set target temperature
- `eightctl alarm list|create|dismiss` - Alarm operations
- `eightctl schedule list|create|update` - Schedule management
- `eightctl audio state|play|pause` - Audio system control
- `eightctl base info|angle` - Base/mattress configuration

### Authentication Methods
- Configuration file: `~/.config/eightctl/config.yaml`
- Environment variables: `EIGHTCTL_EMAIL`, `EIGHTCTL_PASSWORD`
- Session-based authentication with token persistence

## Business Logic

### Authentication and Session Management
1. Authenticate with Eight Sleep API using email/password credentials
2. Obtain and persist session tokens for subsequent operations
3. Handle token refresh and expiration automatically
4. Manage rate limiting and API quota restrictions

### Temperature Control System
1. Validate temperature ranges and safety limits
2. Apply temperature changes gradually to prevent discomfort
3. Monitor temperature sensor feedback and adjustment progress
4. Handle dual-zone temperature control for partner beds

### Sleep Schedule Coordination
1. Create and manage recurring temperature schedules
2. Coordinate schedules with sleep/wake cycles
3. Apply automatic adjustments based on sleep patterns
4. Handle schedule conflicts and priority resolution

### Health and Sleep Tracking
1. Retrieve sleep quality metrics and historical data
2. Monitor heart rate, respiratory rate, and movement
3. Generate sleep insights and trend analysis
4. Export health data for external analysis

## Data Structures

### Pod Status
```typescript
interface PodStatus {
  podId: string;               // device identifier
  isOnline: boolean;           // connectivity status
  isPowered: boolean;          // heating/cooling system state
  currentTemp: number;         // current mattress temperature
  targetTemp: number;          // desired temperature setting
  leftSideTemp?: number;       // left zone temperature (dual-zone)
  rightSideTemp?: number;      // right zone temperature (dual-zone)
  lastUpdate: Date;            // last status update timestamp
  batteryLevel?: number;       // base battery percentage
}
```

### Sleep Data
```typescript
interface SleepSession {
  sessionId: string;           // sleep session identifier
  startTime: Date;             // sleep start timestamp
  endTime?: Date;              // wake time (null if ongoing)
  duration?: number;           // total sleep time in minutes
  deepSleep: number;           // deep sleep duration
  remSleep: number;            // REM sleep duration
  lightSleep: number;          // light sleep duration
  sleepScore: number;          // overall sleep quality score (0-100)
  heartRate: SensorData[];     // heart rate measurements
  respiratoryRate: SensorData[]; // breathing rate data
  movementData: SensorData[];  // bed movement/restlessness
}
```

### Temperature Schedule
```typescript
interface TemperatureSchedule {
  scheduleId: string;          // schedule identifier
  name: string;                // user-friendly schedule name
  isActive: boolean;           // schedule enabled status
  daysOfWeek: number[];        // active days (0=Sunday, 6=Saturday)
  timeSlots: TemperatureSlot[];
  side: 'left' | 'right' | 'both'; // applicable bed side
}

interface TemperatureSlot {
  startTime: string;           // time in HH:MM format
  endTime: string;             // time in HH:MM format
  targetTemp: number;          // temperature setting
  transition: 'gradual' | 'immediate'; // temperature change mode
}
```

### Alarm Configuration
```typescript
interface SleepAlarm {
  alarmId: string;             // alarm identifier
  time: string;                // alarm time in HH:MM format
  daysOfWeek: number[];        // recurring days
  isEnabled: boolean;          // alarm active status
  vibrationIntensity: number;  // vibration level (1-10)
  audioTrack?: string;         // optional audio file
  smartWake: boolean;          // intelligent wake timing
  smartWakeWindow: number;     // wake window in minutes before alarm
}
```

## Constraints

### API and Rate Limiting
- **Unofficial API**: Based on reverse-engineered Eight Sleep API
- **Rate Limits**: Restricted request frequency to prevent account suspension
- **Session Management**: Limited concurrent sessions per account
- **API Stability**: Subject to changes in official Eight Sleep app updates

### Hardware Limitations
- **Temperature Range**: Limited heating/cooling range based on ambient conditions
- **Dual Zone**: Partner beds require coordination between left/right zones
- **Power Requirements**: Base unit must be plugged in and charged
- **Network Dependency**: Requires WiFi connectivity for cloud API access

### Safety and Health Considerations
- **Temperature Limits**: Safety constraints prevent extreme temperature settings
- **Medical Conditions**: Some health conditions may contraindicate temperature therapy
- **Pregnancy Restrictions**: Heating limitations during pregnancy
- **Sleep Disruption**: Frequent temperature changes may disturb sleep quality

## Edge Cases

### Network and Connectivity Issues
- **WiFi Outage**: Pod offline during network connectivity loss
- **API Service Down**: Eight Sleep cloud services unavailable
- **Authentication Failures**: Login credentials expired or invalid
- **Firmware Updates**: Pod temporarily unavailable during firmware updates
- **Rate Limit Exceeded**: Too many API requests causing temporary blocks

### Hardware and Sensor Problems
- **Temperature Sensor Malfunction**: Incorrect temperature readings
- **Base Unit Issues**: Power or charging problems with mattress base
- **Sleep Tracking Failures**: Sensors not detecting presence or movement
- **Dual Zone Conflicts**: Partners setting conflicting temperatures
- **Calibration Drift**: Temperature accuracy degrading over time

### Schedule and Timing Conflicts
- **Overlapping Schedules**: Multiple temperature schedules active simultaneously
- **Time Zone Changes**: Travel affecting local schedule timing
- **Daylight Saving Time**: Schedule timing shifts during DST transitions
- **Power Outage Recovery**: Schedules reset after power loss
- **Schedule Priority**: Determining precedence between manual and automated settings

### Health and Safety Edge Cases
- **Medical Emergency**: Need for immediate temperature adjustment during health episode
- **Sleep Disorder Impact**: Temperature changes affecting sleep disorders
- **Partner Health Differences**: Different health requirements for each bed side
- **Medication Interactions**: Sleep medications affecting temperature regulation
- **Age-related Considerations**: Different temperature needs for elderly users

## Technical Debt

### API Reliability and Maintenance
- Dependency on unofficial API subject to breaking changes
- No service level agreements or support from Eight Sleep
- Limited error handling for undocumented API responses
- No automated testing infrastructure for API changes

### Authentication and Security
- Plain text password storage in configuration files
- No support for two-factor authentication
- Session token management basic and potentially insecure
- Limited audit trail for security-sensitive operations

### User Experience and Safety
- No confirmation prompts for potentially disruptive temperature changes
- Limited validation of safe temperature ranges
- Poor error messages for hardware-related failures
- No rollback mechanism for problematic schedule changes

### Data Management
- No local caching of sleep data for offline analysis
- Limited data export capabilities for health tracking
- No backup mechanism for schedule and preference data
- Basic privacy controls for sensitive health information

## Dependencies

### Core Dependencies
- **eightctl Binary**: Primary CLI tool for Eight Sleep pod control
- **Go Runtime**: Required for eightctl binary execution
- **HTTP Client**: For Eight Sleep API communication

### Authentication Dependencies
- **Eight Sleep Account**: Valid customer account with registered pod
- **Configuration System**: File-based configuration storage
- **Environment Variables**: Alternative credential storage method
- **Session Management**: Token persistence and refresh handling

### Network Dependencies
- **HTTPS Connectivity**: Encrypted communication with Eight Sleep cloud
- **WiFi Infrastructure**: Local network connectivity for pod communication
- **DNS Resolution**: Domain name resolution for API endpoints
- **Certificate Validation**: SSL/TLS certificate chain verification

### Hardware Dependencies
- **Eight Sleep Pod**: Physical smart mattress with base unit
- **Power Supply**: Electrical power for base unit and sensors
- **Sensor Array**: Temperature, movement, and biometric sensors
- **Wireless Connectivity**: WiFi module in base unit for cloud communication

### System Dependencies
- **File System**: Configuration file storage and access
- **Process Management**: Command execution and session handling
- **Time/Date**: Accurate system time for scheduling operations
- **Logging**: Error and operation logging capabilities

### Optional Dependencies
- **Health Apps**: Integration with fitness/health tracking platforms
- **Home Automation**: Smart home integration (HomeKit, Alexa, etc.)
- **Sleep Apps**: Third-party sleep tracking and analysis tools
- **Medical Systems**: Integration with healthcare monitoring platforms