# Config Module Specification

## Intent
The Config module provides centralized configuration management for the entire OpenClaw system. It handles configuration file loading, validation, schema enforcement, hot-reloading, and migration between configuration versions. The module ensures type-safe configuration access, provides default value resolution, and manages the complex inheritance hierarchy between global defaults, agent-specific overrides, and runtime parameters.

## Interface

### Core Types
```typescript
export type OpenClawConfig = {
  agents?: AgentsConfig;
  channels?: ChannelsConfig;
  providers?: ProvidersConfig;
  tools?: ToolsConfig;
  gateway?: GatewayConfig;
  memory?: MemoryConfig;
  sandbox?: SandboxConfig;
  logging?: LoggingConfig;
  update?: UpdateConfig;
}

export type ConfigIO = {
  read: (path: string) => Promise<any>;
  write: (path: string, data: any) => Promise<void>;
  exists: (path: string) => Promise<boolean>;
  watch: (path: string, callback: (event: string) => void) => void;
}
```

### Key Functions
- `loadConfig(path?: string)` - Load configuration from file with validation
- `writeConfigFile(path, config)` - Save configuration with atomic writes
- `validateConfigObjectWithPlugins(config)` - Validate against schema
- `applyLegacyMigrations(config)` - Migrate old configuration formats
- `resolveConfigPath(filename?)` - Resolve configuration file path
- `createConfigIO(options)` - Create configuration I/O interface

### Configuration Resolution
- **File Discovery**: Automatic config file location and format detection
- **Schema Validation**: Comprehensive validation with detailed error reporting
- **Default Merging**: Intelligent merging of defaults with user overrides
- **Environment Substitution**: Environment variable expansion in config values
- **Hot Reloading**: Live configuration updates without service restart

## Business Logic

### Configuration Loading Process
1. **Path Resolution**: Locate config file using standard search paths
2. **Format Detection**: Auto-detect YAML, JSON, or JSON5 format
3. **File Reading**: Load raw configuration with error handling
4. **Environment Expansion**: Replace environment variable references
5. **Schema Validation**: Validate against comprehensive Zod schemas
6. **Default Merging**: Apply system defaults for missing values
7. **Migration**: Apply any necessary format migrations
8. **Caching**: Store validated config in memory cache

### Validation Framework
- **Zod Schemas**: Type-safe validation with detailed error messages
- **Plugin Validation**: Extended validation for plugin-specific config
- **Cross-field Validation**: Validate relationships between config sections
- **Warning Collection**: Non-fatal validation warnings for deprecated options
- **Custom Validators**: Extensible validation for special requirements

### Hot Reload System
- **File Watching**: Monitor config files for changes
- **Debouncing**: Prevent excessive reloads from rapid file changes
- **Validation Before Apply**: Ensure new config is valid before switching
- **Rollback Capability**: Revert to previous config on validation failure
- **Notification**: Notify dependent systems of configuration changes

### Migration System
- **Version Detection**: Identify configuration format versions
- **Backward Compatibility**: Support for legacy configuration formats
- **Field Renaming**: Automatic migration of renamed configuration fields
- **Structure Changes**: Handle configuration structure reorganization
- **Data Transformation**: Convert between different value formats

## Data Structures

### Configuration Cache
```typescript
interface ConfigCache {
  raw: any;                    // Raw loaded configuration
  validated: OpenClawConfig;   // Validated and processed configuration
  hash: string;                // Configuration content hash
  timestamp: number;           // Last modification time
  errors: ValidationError[];   // Validation errors if any
  warnings: ValidationWarning[]; // Non-fatal validation warnings
}
```

### Schema Registry
```typescript
interface SchemaRegistry {
  core: ZodSchema;             // Core configuration schema
  plugins: Map<string, ZodSchema>; // Plugin-specific schemas
  validators: CustomValidator[]; // Custom validation functions
  migrations: MigrationStep[]; // Schema migration steps
}
```

### Configuration Watcher
```typescript
interface ConfigWatcher {
  path: string;
  watcher: FSWatcher;
  debounceTimer?: NodeJS.Timeout;
  lastHash: string;
  onChange: (config: OpenClawConfig) => void;
  onError: (error: Error) => void;
}
```

### Default Values Registry
```typescript
interface DefaultRegistry {
  agents: Partial<AgentDefaultsConfig>;
  channels: Partial<ChannelDefaultsConfig>;
  tools: Partial<ToolsConfig>;
  gateway: Partial<GatewayConfig>;
  providers: Partial<ProvidersConfig>;
  [key: string]: any;
}
```

## Constraints

### File Format Support
- **YAML**: Primary configuration format with full feature support
- **JSON5**: Extended JSON with comments and trailing commas
- **JSON**: Standard JSON with limited comment support
- **Maximum Size**: 10MB configuration file size limit
- **Encoding**: UTF-8 encoding required for all configuration files

### Validation Performance
- **Schema Compile Time**: Zod schema compilation under 100ms
- **Validation Time**: Configuration validation under 50ms
- **Memory Usage**: Configuration cache limited to 50MB
- **File Watch Overhead**: Minimal impact on file system performance
- **Error Collection**: Maximum 100 validation errors per config load

### Configuration Complexity
- **Nesting Depth**: Maximum 10 levels of configuration nesting
- **Array Size**: Maximum 1000 items in configuration arrays
- **String Length**: Maximum 10KB for individual configuration strings
- **Environment Variables**: Maximum 500 environment variable references
- **Plugin Config**: Maximum 100 plugin-specific configuration sections

## Edge Cases

### File System Issues
- **Missing Config File**: Use defaults and create template configuration
- **Permission Errors**: Fallback to read-only mode with warning
- **Disk Full**: Handle write failures gracefully with error reporting
- **File Corruption**: Detect and recover from malformed configuration files
- **Concurrent Writes**: Handle file locking and write conflicts

### Validation Failures
- **Schema Violations**: Provide detailed error messages with field paths
- **Type Mismatches**: Clear error messages for type conversion failures
- **Missing Required Fields**: Helpful guidance on required configuration
- **Invalid References**: Validate cross-references between config sections
- **Circular Dependencies**: Detect and report circular configuration references

### Hot Reload Complications
- **Partial Updates**: Handle incomplete configuration file changes
- **Validation Errors**: Maintain current config when new config is invalid
- **Dependent Service Failures**: Handle services that fail to update
- **Race Conditions**: Prevent concurrent configuration updates
- **Resource Cleanup**: Ensure proper cleanup of old configuration resources

### Migration Edge Cases
- **Version Ambiguity**: Handle configurations with unclear versions
- **Migration Failures**: Rollback to original configuration on migration errors
- **Data Loss**: Prevent configuration data loss during migrations
- **Incompatible Changes**: Handle breaking changes in configuration format
- **Multiple Migrations**: Apply sequential migrations correctly

## Technical Debt

### Current Issues
- **Configuration Complexity**: Deeply nested configuration structure
- **Default Resolution**: Complex default value inheritance logic
- **Error Messages**: Cryptic validation error messages for end users
- **Schema Maintenance**: Keeping Zod schemas synchronized with TypeScript types
- **Performance**: Repeated validation of unchanged configuration sections

### Design Problems
- **Global State**: Configuration stored in global variables
- **Tight Coupling**: Direct dependencies between configuration sections
- **Validation Timing**: Validation performed too late in startup process
- **Error Recovery**: Limited error recovery options for invalid configurations
- **Testing**: Difficult to test configuration loading edge cases

### Missing Features
- **Configuration Templates**: Predefined configuration templates for common setups
- **Interactive Configuration**: CLI wizard for configuration generation
- **Configuration Documentation**: Auto-generated documentation from schemas
- **Configuration Linting**: Static analysis for common configuration mistakes
- **Visual Configuration**: Web-based configuration editor

### Legacy Support
- **Old Format Support**: Maintaining compatibility with multiple old formats
- **Deprecated Fields**: Handling deprecated configuration fields gracefully
- **Migration Testing**: Comprehensive testing of migration scenarios
- **Version Detection**: Reliable detection of configuration format versions
- **Documentation**: Keeping migration documentation updated

## Dependencies

### Core Dependencies
- **zod**: Schema validation and type inference
- **json5**: JSON5 configuration file parsing
- **js-yaml**: YAML configuration file parsing
- **chokidar**: File system watching for hot reload
- **lodash**: Deep merging and object manipulation

### Internal Dependencies
- **plugin-sdk**: Plugin system integration for extended validation
- **utils**: File path resolution and utility functions
- **logging**: Configuration change logging and debugging
- **errors**: Centralized error handling and formatting
- **types**: TypeScript type definitions for configuration

### File System Dependencies
- **fs/promises**: Asynchronous file operations
- **path**: Cross-platform path manipulation
- **os**: Operating system specific configuration paths
- **process**: Environment variable access
- **worker_threads**: Background configuration processing

### Optional Dependencies
- **ajv**: Alternative JSON schema validation (legacy support)
- **typescript**: Runtime type checking for development mode
- **prettier**: Configuration file formatting
- **eslint**: Configuration file linting
- **json-schema**: JSON schema generation from Zod schemas