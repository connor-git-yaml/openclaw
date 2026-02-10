# Heartbeat Module Specification

## Intent
The Heartbeat module provides proactive agent health monitoring and periodic task execution capabilities for OpenClaw. It enables agents to perform regular checks, maintenance tasks, and proactive notifications without explicit user prompting. The module supports configurable heartbeat intervals, task scheduling, health monitoring, and integration with user-defined automation workflows.

## Interface

### Core Types
```typescript
export type HeartbeatConfig = {
  enabled: boolean;
  intervalMs: number;
  promptTemplate: string;
  maxConsecutiveErrors: number;
  backoffMultiplier: number;
  tasks: HeartbeatTask[];
  healthChecks: HealthCheck[];
};

export type HeartbeatTask = {
  id: string;
  name: string;
  enabled: boolean;
  intervalMs: number;
  lastRun?: Date;
  nextRun?: Date;
  maxRetries: number;
  retryCount: number;
  condition?: string;
  action: TaskAction;
};

export type HealthCheck = {
  id: string;
  name: string;
  type: 'system' | 'service' | 'network' | 'storage';
  enabled: boolean;
  intervalMs: number;
  timeout: number;
  thresholds: Record<string, number>;
  lastCheck?: Date;
  status: 'healthy' | 'warning' | 'critical' | 'unknown';
};

export type TaskAction = {
  type: 'check_email' | 'check_calendar' | 'check_weather' | 'check_mentions' | 
        'update_memory' | 'cleanup_files' | 'custom_script';
  params: Record<string, any>;
};
```

### Key Functions
- `startHeartbeat(agentId, config)` - Start heartbeat monitoring for agent
- `stopHeartbeat(agentId)` - Stop heartbeat monitoring
- `scheduleTask(task)` - Schedule periodic task execution
- `executeHeartbeat(agentId)` - Manually trigger heartbeat execution
- `addHealthCheck(check)` - Add health monitoring check
- `getHeartbeatStatus(agentId)` - Get current heartbeat status
- `updateHeartbeatConfig(agentId, config)` - Update heartbeat configuration

### Default Heartbeat Actions
- **Email Check**: Monitor for new important emails
- **Calendar Check**: Upcoming events and reminders
- **Weather Check**: Daily weather updates if relevant
- **Mention Monitoring**: Social media and notification monitoring
- **Memory Maintenance**: Update long-term memory files
- **File Cleanup**: Clean up temporary files and logs

## Business Logic

### Heartbeat Execution Cycle
1. **Schedule Management**: Maintain schedule of next heartbeat executions
2. **Context Preparation**: Read HEARTBEAT.md file for current priorities
3. **Task Evaluation**: Determine which tasks need execution
4. **Condition Checking**: Evaluate task conditions and triggers
5. **Execution**: Run enabled tasks with proper error handling
6. **Result Processing**: Handle task results and update state
7. **Health Monitoring**: Perform system health checks
8. **Scheduling Update**: Calculate next execution time

### Proactive Task Management
- **Email Monitoring**: Check for urgent or important emails
- **Calendar Integration**: Upcoming event notifications and reminders
- **System Health**: Monitor system resources and service status
- **File Maintenance**: Automatic cleanup and organization tasks
- **Memory Updates**: Periodic memory file review and updates
- **Background Optimization**: Performance optimization tasks

### Intelligent Scheduling
- **Adaptive Intervals**: Adjust frequency based on activity patterns
- **Priority-Based Execution**: Execute high-priority tasks first
- **Condition-Based Triggers**: Execute tasks when specific conditions are met
- **Backoff Strategy**: Reduce frequency after repeated failures
- **User Context Awareness**: Avoid interruptions during focus time

### Health Monitoring System
- **System Resources**: Monitor CPU, memory, disk usage
- **Service Availability**: Check critical service health
- **Network Connectivity**: Monitor network connectivity and latency
- **Storage Health**: Monitor disk space and file system health
- **Error Rate Tracking**: Track and alert on error rate spikes

### Error Handling and Recovery
- **Graceful Degradation**: Continue operation even with failed tasks
- **Retry Logic**: Automatic retry with exponential backoff
- **Error Reporting**: Report persistent errors to user
- **Circuit Breaker**: Disable failing tasks temporarily
- **Recovery Procedures**: Automatic recovery from common failures

## Data Structures

### Heartbeat State
```typescript
type HeartbeatState = {
  agentId: string;
  enabled: boolean;
  lastHeartbeat: Date;
  nextHeartbeat: Date;
  consecutiveErrors: number;
  totalExecutions: number;
  tasks: Map<string, TaskState>;
  healthChecks: Map<string, HealthCheckState>;
};
```

### Task State
```typescript
type TaskState = {
  id: string;
  enabled: boolean;
  lastRun?: Date;
  nextRun: Date;
  retryCount: number;
  lastResult?: TaskResult;
  averageExecutionTime: number;
  errorHistory: TaskError[];
};
```

### Health Check State
```typescript
type HealthCheckState = {
  id: string;
  status: 'healthy' | 'warning' | 'critical' | 'unknown';
  lastCheck: Date;
  lastValue?: number;
  history: HealthCheckResult[];
  alertsSent: number;
  muted: boolean;
};
```

### Task Result
```typescript
type TaskResult = {
  success: boolean;
  executionTime: number;
  output?: any;
  error?: string;
  timestamp: Date;
  nextAction?: 'continue' | 'pause' | 'disable';
};
```

## Constraints

### Timing Constraints
- **Minimum Interval**: 30 seconds minimum between heartbeats
- **Maximum Interval**: 24 hours maximum between heartbeats
- **Execution Timeout**: 60 seconds maximum per heartbeat execution
- **Task Timeout**: 30 seconds maximum per individual task

### Resource Constraints
- **Concurrent Tasks**: Maximum 5 concurrent heartbeat tasks
- **Memory Usage**: Heartbeat state limited to 10MB per agent
- **Storage**: Log files limited to 100MB total size
- **CPU Usage**: Heartbeat tasks limited to 5% CPU usage

### User Experience Constraints
- **Quiet Hours**: Respect user-defined quiet hours (23:00-08:00)
- **Focus Mode**: Reduce frequency during active user sessions
- **Notification Limits**: Maximum 3 proactive notifications per hour
- **Battery Awareness**: Reduce frequency on battery-powered devices

### Platform Limitations
- **Mobile Restrictions**: Limited background execution on mobile platforms
- **Network Dependencies**: Some tasks require network connectivity
- **Permission Requirements**: Some health checks require elevated permissions
- **Service Availability**: External service dependencies may be unavailable

## Edge Cases

### Network Connectivity Issues
- **Offline Operation**: Continue essential tasks without network
- **Intermittent Connectivity**: Handle flaky network connections
- **API Rate Limits**: Respect external service rate limits
- **Service Outages**: Graceful handling of external service failures

### System Resource Exhaustion
- **High CPU Usage**: Defer tasks during high system load
- **Low Memory**: Reduce memory usage during memory pressure
- **Low Disk Space**: Prioritize cleanup tasks when disk space is low
- **Network Congestion**: Reduce network-intensive tasks

### Configuration Edge Cases
- **Invalid Configuration**: Handle malformed heartbeat configuration
- **Circular Dependencies**: Detect and prevent task circular dependencies
- **Schedule Conflicts**: Resolve overlapping task schedules
- **Permission Changes**: Handle changes in system permissions

### Error Recovery Scenarios
- **Persistent Failures**: Handle tasks that repeatedly fail
- **Partial Failures**: Handle cases where some tasks succeed and others fail
- **Recovery After Downtime**: Proper state recovery after system restart
- **Clock Changes**: Handle system time changes and timezone updates

## Technical Debt

### Known Issues
- **State Persistence**: Heartbeat state is not persisted across restarts
- **Task Dependencies**: No support for task dependencies and ordering
- **Dynamic Configuration**: Configuration changes require restart
- **Error Reporting**: Limited error reporting and debugging information

### Performance Optimizations Needed
- **Task Batching**: Batch similar tasks for efficiency
- **Lazy Loading**: Load task modules only when needed
- **Caching**: Cache expensive operations between heartbeats
- **Resource Pooling**: Reuse resources across task executions

### Feature Gaps
- **Task Templating**: No templating system for common task patterns
- **Conditional Execution**: Limited condition evaluation capabilities
- **External Triggers**: No support for external event-driven triggers
- **Analytics**: No analytics or reporting for heartbeat performance

### Monitoring Improvements Needed
- **Health Dashboards**: Visual dashboard for heartbeat health
- **Alerting System**: Configurable alerting for health check failures
- **Performance Metrics**: Detailed performance metrics and trending
- **Log Aggregation**: Centralized logging for distributed heartbeat instances

## Dependencies

### Internal Dependencies
- **Agents**: Integration with agent execution context
- **Config**: Heartbeat configuration and user preferences
- **Tools**: Access to tool system for task execution
- **Sessions**: Agent session management and lifecycle

### External Dependencies
- **Scheduler**: Timing and scheduling library (node-cron or similar)
- **File System**: Access to HEARTBEAT.md and configuration files
- **Network**: HTTP client for external service monitoring
- **System Monitoring**: OS-specific system monitoring libraries

### Platform Dependencies
- **macOS**: System monitoring via IOKit and ActivityMonitor
- **Linux**: System monitoring via /proc filesystem and systemd
- **Windows**: System monitoring via WMI and Performance Counters
- **Node.js**: Timer and event loop integration for scheduling