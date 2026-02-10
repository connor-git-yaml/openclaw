# Healthcheck Skill Specification

## Intent

Provide comprehensive security assessment and hardening capabilities for OpenClaw host systems. Implements risk-based security posture evaluation with automated remediation, periodic monitoring, and compliance verification for workstations, servers, and containerized deployments.

## Interface

### Core Assessment Operations
- **Security Audit**: Comprehensive host security evaluation using OpenClaw security tools
- **Risk Profiling**: User-defined risk tolerance configuration and gap analysis
- **System Context**: OS, network exposure, access methods, and backup status evaluation
- **Hardening Plans**: Step-by-step remediation with rollback strategies

### Workflow Structure
1. **Model Validation**: Ensure state-of-the-art AI model for security decisions
2. **Context Discovery**: Read-only system assessment and environment mapping
3. **OpenClaw Audits**: Built-in security audit execution and analysis
4. **Risk Tolerance**: User-defined security posture selection
5. **Remediation Planning**: Detailed hardening plan generation
6. **Guided Execution**: Step-by-step implementation with confirmations
7. **Verification**: Post-hardening validation and reporting

### Command Integration
- `openclaw security audit [--deep] [--fix] [--json]` - Security assessment
- `openclaw update status` - Version and update checking
- `openclaw cron add/list/runs` - Periodic monitoring scheduling
- OS-specific commands for firewall, SSH, and service management

## Business Logic

### Security Assessment Framework
1. Execute comprehensive read-only system discovery
2. Analyze network exposure, service configuration, and access patterns
3. Evaluate current security posture against industry standards
4. Identify vulnerabilities and configuration weaknesses
5. Generate risk-prioritized finding reports with remediation guidance

### Risk-Based Hardening Strategy
1. Establish user risk tolerance through guided questionnaire
2. Map current security state to target security posture
3. Generate remediation plans with minimal disruption
4. Preserve access methods and operational requirements
5. Implement defense-in-depth with layered security controls

### Change Management and Safety
1. Require explicit approval for all state-changing operations
2. Provide detailed command explanations and impact analysis
3. Implement rollback procedures for each hardening step
4. Validate access preservation before proceeding
5. Document all changes with audit trail and reasoning

### Continuous Monitoring Integration
1. Establish baseline security measurements after hardening
2. Schedule periodic audits via OpenClaw cron system
3. Monitor for configuration drift and new vulnerabilities
4. Provide automated reporting and alerting capabilities
5. Support update status monitoring and version tracking

## Data Structures

### System Context
```typescript
interface SystemContext {
  os: {
    type: 'linux' | 'macos' | 'windows';
    version: string;
    isContainer: boolean;
  };
  access: {
    privilegeLevel: 'root' | 'admin' | 'user';
    connectionMethod: 'local' | 'ssh' | 'rdp' | 'tailnet';
    networkExposure: 'public' | 'lan' | 'isolated';
  };
  services: {
    openclawGateway: GatewayStatus;
    listeningPorts: ServicePort[];
    firewallStatus: FirewallConfig;
  };
  security: {
    diskEncryption: boolean;
    autoUpdates: boolean;
    backupStatus: BackupInfo;
  };
  deploymentContext: 'workstation' | 'server' | 'container' | 'ci';
}
```

### Risk Profile
```typescript
interface RiskProfile {
  name: string;               // profile identifier
  description: string;        // risk tolerance description
  allowedServices: string[];  // permitted network services
  firewallPolicy: 'deny-default' | 'allow-default' | 'custom';
  remoteAccess: 'disabled' | 'lan-only' | 'vpn-only' | 'restricted';
  updatePolicy: 'automatic' | 'manual' | 'delayed';
  monitoringLevel: 'basic' | 'enhanced' | 'comprehensive';
  complianceRequirements: string[];
}
```

### Security Audit Result
```typescript
interface SecurityAuditResult {
  timestamp: Date;
  version: string;            // OpenClaw version
  findings: SecurityFinding[];
  overallScore: number;       // 0-100 security score
  criticalIssues: number;
  recommendedActions: string[];
  complianceStatus: ComplianceCheck[];
}

interface SecurityFinding {
  id: string;
  severity: 'critical' | 'high' | 'medium' | 'low' | 'info';
  title: string;
  description: string;
  impact: string;
  remediation: string;
  category: 'network' | 'access' | 'encryption' | 'configuration';
}
```

### Hardening Plan
```typescript
interface HardeningPlan {
  planId: string;             // unique plan identifier
  timestamp: Date;
  targetProfile: RiskProfile;
  currentState: SystemContext;
  gapAnalysis: SecurityGap[];
  remediationSteps: HardeningStep[];
  rollbackProcedures: RollbackStep[];
  estimatedDuration: number;  // minutes
  riskAssessment: string;
}

interface HardeningStep {
  stepId: number;
  title: string;
  description: string;
  commands: string[];
  expectedOutput: string;
  verificationSteps: string[];
  rollbackCommands: string[];
  accessImpact: string;
  approvalRequired: boolean;
}
```

### Monitoring Configuration
```typescript
interface MonitoringConfig {
  scheduleId: string;
  jobType: 'security-audit' | 'update-status';
  frequency: 'daily' | 'weekly' | 'monthly';
  timeWindow: string;         // preferred execution time
  outputLocation: string;     // results storage path
  alertThresholds: AlertThreshold[];
  retentionDays: number;
}
```

## Constraints

### Model and Capability Requirements
- **AI Model Standards**: Recommends state-of-the-art models (Opus 4.5+, GPT 5.2+) for security decisions
- **Security Expertise**: Requires advanced understanding of system hardening principles
- **Platform Coverage**: Must support Linux, macOS, and Windows security models
- **Command Accuracy**: Limited to verified OpenClaw and OS commands

### Safety and Change Management
- **Explicit Approval**: All state-changing operations require user confirmation
- **Access Preservation**: Must not break existing access methods without user consent
- **Reversibility**: All hardening steps must include rollback procedures
- **Documentation**: Changes require comprehensive audit trail and justification

### System Integration Constraints
- **Read-Only Discovery**: Initial assessment must not modify system state
- **Privilege Boundaries**: Respects user privilege levels and security boundaries
- **Service Dependencies**: Must not break OpenClaw or other critical services
- **Network Requirements**: Hardening must preserve required network connectivity

## Edge Cases

### Access and Authentication Issues
- **Remote Access Lockout**: Hardening SSH/RDP settings preventing reconnection
- **Firewall Misconfigurations**: Rules blocking essential services or management access
- **Certificate Issues**: SSL/TLS configuration problems affecting service access
- **User Account Lockout**: Permission changes preventing normal user operations
- **Service Account Issues**: Breaking service authentication after hardening

### Platform and Environment Complications
- **Container Limitations**: Security controls not applicable in containerized environments
- **Cloud Platform Restrictions**: Hosted environments with limited configuration access
- **Corporate Policy Conflicts**: Hardening recommendations conflicting with organizational policies
- **Legacy System Constraints**: Older systems unable to support modern security practices
- **Multi-User Conflicts**: Hardening affecting other users on shared systems

### Monitoring and Maintenance Challenges
- **Cron Job Failures**: Scheduled security audits failing due to system changes
- **Storage Issues**: Audit logs consuming excessive disk space
- **Performance Impact**: Security monitoring affecting system performance
- **False Positive Alerts**: Security tools generating spurious warnings
- **Update Conflicts**: Automatic updates interfering with hardening configurations

### Recovery and Rollback Problems
- **Incomplete Rollbacks**: Partial reversal of hardening changes
- **Configuration Drift**: System changes invalidating rollback procedures
- **Backup Failures**: Unable to restore previous configuration state
- **Documentation Loss**: Missing information for troubleshooting hardening issues
- **Time Window Constraints**: Rollback procedures taking longer than expected

## Technical Debt

### Assessment Coverage Gaps
- Limited support for specialized environments (IoT, embedded systems)
- Basic compliance framework without industry-specific standards
- No integration with external security scanning tools
- Insufficient coverage of application-layer security controls

### Automation and Orchestration
- Manual step-by-step execution instead of automated workflows
- No integration with configuration management tools
- Limited support for infrastructure-as-code practices
- Basic scheduling without advanced workflow orchestration

### Monitoring and Alerting Limitations
- Simple threshold-based alerting without intelligent analysis
- No integration with external security information systems
- Basic reporting without trend analysis or predictive capabilities
- Limited correlation of security events across time

### Documentation and Knowledge Management
- Security findings not integrated with organizational knowledge base
- Limited sharing of hardening configurations across similar systems
- No templates for common deployment patterns
- Insufficient integration with change management processes

## Dependencies

### Core OpenClaw Dependencies
- **OpenClaw Security Framework**: Built-in security audit capabilities
- **OpenClaw Cron System**: Scheduling infrastructure for periodic monitoring
- **OpenClaw Update System**: Version checking and update status reporting
- **OpenClaw Configuration**: Access to gateway and service configuration

### Operating System Dependencies
- **System Administration Tools**: Platform-specific configuration utilities
- **Firewall Management**: iptables, UFW, Windows Firewall, pfctl
- **Service Management**: systemd, launchd, Windows Services
- **Security Tools**: Platform-native security assessment utilities

### Network and Access Dependencies
- **SSH/RDP Clients**: Remote access tools for verification and rollback
- **Network Analysis**: Port scanning and connectivity testing tools
- **Certificate Management**: SSL/TLS certificate validation and management
- **VPN/Tunneling**: Secure access mechanisms for remote systems

### Monitoring and Logging Dependencies
- **System Logging**: Platform logging infrastructure for audit trails
- **File System**: Persistent storage for configuration and audit data
- **Process Management**: Background task execution for monitoring
- **Notification System**: Alerting mechanisms for security events

### Optional Dependencies
- **Backup Systems**: Time Machine, system snapshots, configuration backups
- **Compliance Tools**: Industry-specific security assessment frameworks
- **External Scanners**: Integration with vulnerability scanning platforms
- **Configuration Management**: Ansible, Puppet, Chef for large-scale deployment

### Development and Maintenance Dependencies
- **Security Knowledge Base**: Current threat intelligence and best practices
- **Platform Documentation**: OS-specific security configuration guidance
- **Testing Framework**: Validation of hardening procedures in test environments
- **Community Resources**: Security community input and peer review