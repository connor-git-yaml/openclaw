# Model Usage Skill Specification

## Intent

Provide AI model usage and cost analysis through CodexBar CLI integration. Enables tracking, monitoring, and reporting of per-model expenses for Codex and Claude usage patterns with current model identification and historical analysis capabilities.

## Interface

### Core Operations
- **Current Model Usage**: Identify and report on most recent model usage costs
- **Historical Analysis**: Comprehensive per-model cost breakdowns over time
- **Provider-Specific Reporting**: Separate analysis for Codex and Claude providers
- **Cost Summarization**: Aggregated usage statistics and trend analysis

### Command Structure
- `python model_usage.py --provider codex --mode current` - Current model costs
- `python model_usage.py --provider claude --mode all` - All models historical
- `python model_usage.py --input FILE --mode current` - File-based analysis
- `python model_usage.py --format json --pretty` - Structured output

### Input Methods
- **Live Data**: Direct CodexBar CLI integration for real-time usage
- **File Input**: Analysis of exported JSON cost data
- **Stdin Processing**: Pipeline integration for automated workflows

### Output Formats
- Human-readable text summaries with cost breakdowns
- JSON format (`--format json --pretty`) for programmatic processing
- Model-specific reporting with usage patterns

## Business Logic

### Cost Data Collection and Processing
1. Interface with CodexBar CLI to retrieve provider-specific cost data
2. Parse JSON usage logs with model breakdown information
3. Handle missing or incomplete cost data gracefully
4. Aggregate usage statistics across time periods

### Current Model Identification Logic
1. Locate most recent daily log entry with model breakdown data
2. Identify model with highest cost in recent time period
3. Fall back to last model in usage history when breakdowns unavailable
4. Support manual model specification for targeted analysis

### Historical Usage Analysis
1. Process time-series cost data for trend identification
2. Aggregate per-model usage across multiple time periods
3. Calculate usage patterns and cost efficiency metrics
4. Generate comparative analysis between different models

### Provider-Specific Processing
1. Handle Codex and Claude provider data formats
2. Account for provider-specific cost structures and billing models
3. Normalize data across different AI service providers
4. Support provider-agnostic reporting and comparison

## Data Structures

### Usage Summary
```typescript
interface UsageSummary {
  provider: 'codex' | 'claude';     // AI service provider
  mode: 'current' | 'all';          // analysis scope
  timeRange: {
    start: Date;
    end: Date;
  };
  totalCost: number;                 // aggregate cost
  modelBreakdown: ModelUsage[];      // per-model details
  currentModel?: ModelUsage;         // most recent model
}
```

### Model Usage Entry
```typescript
interface ModelUsage {
  modelName: string;                 // AI model identifier
  totalCost: number;                 // cumulative cost
  usageCount: number;                // number of API calls
  averageCost: number;               // cost per call
  firstUsed: Date;                   // initial usage timestamp
  lastUsed: Date;                    // most recent usage
  costTrend: TrendAnalysis;          // usage pattern analysis
}
```

### CodexBar Cost Data
```typescript
interface CodexBarCostData {
  date: string;                      // daily log date
  provider: string;                  // service provider
  totalCost: number;                 // daily total cost
  modelsUsed: string[];              // models used list
  modelBreakdowns?: ModelBreakdown[]; // detailed per-model costs
  tokenCount?: number;               // total tokens processed
}

interface ModelBreakdown {
  model: string;                     // model name
  cost: number;                      // model-specific cost
  percentage: number;                // cost percentage of total
}
```

### Analysis Configuration
```typescript
interface AnalysisConfig {
  provider: 'codex' | 'claude';     // target provider
  mode: 'current' | 'all';          // analysis mode
  inputSource: 'cli' | 'file' | 'stdin'; // data source
  inputPath?: string;                // file path for file input
  outputFormat: 'text' | 'json';    // output format
  pretty: boolean;                   // formatted JSON output
  modelOverride?: string;            // specific model filter
  dateRange?: {                      // time range filter
    start: Date;
    end: Date;
  };
}
```

### Trend Analysis
```typescript
interface TrendAnalysis {
  direction: 'increasing' | 'decreasing' | 'stable';
  changeRate: number;                // percentage change
  peakUsage: Date;                   // highest usage date
  averageDailyCost: number;          // daily cost average
  projectedMonthlyCost: number;      // cost projection
}
```

## Constraints

### Platform Dependencies
- **macOS Requirement**: CodexBar CLI currently limited to macOS platform
- **CodexBar Installation**: Must have CodexBar application installed via brew cask
- **Python Runtime**: Requires Python environment for analysis scripts
- **CLI Access**: CodexBar must expose command-line interface

### Data Format Limitations
- **JSON Structure**: Dependent on CodexBar's specific JSON format
- **Model Breakdown Availability**: Not all cost entries include detailed model breakdowns
- **Token Limitations**: Token counts not always split by model in source data
- **Provider Coverage**: Limited to Codex and Claude providers only

### Cost Data Accuracy
- **Local Logging**: Relies on CodexBar's local cost tracking accuracy
- **API Synchronization**: Cost data may lag behind actual API usage
- **Model Mapping**: Model names must match CodexBar's internal naming
- **Currency Assumptions**: Cost calculations assume consistent currency units

## Edge Cases

### Data Availability Problems
- **Missing Cost Data**: CodexBar logs empty or corrupted
- **Incomplete Model Breakdowns**: Recent entries lacking detailed model information
- **Provider Data Gaps**: Specific provider data unavailable or incomplete
- **Time Range Issues**: Requested analysis period outside available data range

### Analysis Logic Complications
- **Multiple Models Same Cost**: Ambiguous "highest cost" model identification
- **Zero Usage Periods**: No model usage in specified time range
- **Model Name Changes**: AI service provider changing model naming conventions
- **Cost Structure Changes**: Provider billing model modifications affecting calculations

### Input and Output Issues
- **File Format Errors**: Malformed JSON input data
- **Permission Problems**: Unable to read input files or write output
- **Pipeline Failures**: Stdin processing interruption or corruption
- **Large Dataset Performance**: Memory issues with extensive historical data

### Configuration and Environment Issues
- **CodexBar CLI Unavailable**: Application not installed or CLI not functional
- **Python Environment Problems**: Missing dependencies or incompatible versions
- **Path Resolution Issues**: Script execution from incorrect working directory
- **Environment Variable Conflicts**: System configuration affecting script execution

## Technical Debt

### Platform Portability Limitations
- macOS-only support limiting cross-platform deployment
- No documented Linux CLI installation path
- Windows platform not supported for CodexBar integration
- No alternative data sources for non-macOS platforms

### Data Processing Robustness
- Basic error handling for malformed or missing data
- Limited validation of CodexBar JSON format consistency
- No automatic recovery from incomplete or corrupted logs
- Simple fallback logic when model breakdowns unavailable

### Analysis Capabilities
- Basic statistical analysis without advanced trend modeling
- No predictive analytics or cost forecasting features
- Limited comparative analysis between providers
- No integration with external cost management systems

### User Experience and Automation
- Command-line only interface without graphical components
- Manual script execution without automated scheduling
- Basic output formatting without customizable report templates
- Limited integration with monitoring and alerting systems

## Dependencies

### Core Dependencies
- **CodexBar Application**: macOS menu bar app for AI usage tracking
- **CodexBar CLI**: Command-line interface for data export
- **Python Runtime**: Script execution environment
- **JSON Processing**: Data parsing and manipulation libraries

### System Dependencies
- **macOS Platform**: Operating system requirement for CodexBar
- **Homebrew**: Package manager for CodexBar installation
- **File System Access**: Read permissions for cost data and write for output
- **Command Execution**: Shell environment for CLI invocation

### Data Dependencies
- **Cost Tracking Logs**: CodexBar's local usage and cost database
- **Provider APIs**: Underlying Codex and Claude service integration
- **Model Metadata**: AI model naming and identification information
- **Billing Data**: Accurate cost information from AI service providers

### Analysis Dependencies
- **Python Standard Library**: JSON, datetime, and data processing modules
- **Statistical Libraries**: Basic mathematical operations for trend analysis
- **Text Processing**: Output formatting and report generation
- **Error Handling**: Exception management and graceful degradation

### Optional Dependencies
- **Advanced Analytics**: Enhanced statistical analysis libraries
- **Visualization**: Graphical chart generation for usage trends
- **Database Storage**: Persistent storage for historical analysis
- **Notification Integration**: Alerting systems for cost threshold monitoring