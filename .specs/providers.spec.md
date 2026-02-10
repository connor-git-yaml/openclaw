# Providers Module Specification

## Intent
The Providers module manages integration with AI model providers (OpenAI, Anthropic, Google, etc.) and handles model authentication, request routing, response processing, and usage tracking. It abstracts the complexities of different provider APIs into a unified interface, enabling seamless switching between models and providers while maintaining consistent behavior across the OpenClaw ecosystem.

## Interface

### Core Types
```typescript
export type ProviderConfig = {
  [providerId: string]: ProviderSettings;
}

export type ProviderSettings = {
  enabled?: boolean;
  apiKey?: string;
  baseUrl?: string;
  models?: string[];
  defaultModel?: string;
  rateLimits?: RateLimitConfig;
  headers?: Record<string, string>;
  timeout?: number;
}

export type ModelRef = {
  provider: string;
  model: string;
  capabilities?: string[];
  contextLength?: number;
  costPer1kTokens?: number;
}
```

### Key Functions
- `resolveModelProvider(modelRef)` - Get provider instance for model reference
- `callModel(provider, model, messages, options)` - Execute model API call
- `validateProviderAuth(providerId)` - Verify provider authentication
- `getProviderCapabilities(providerId)` - Query provider feature support
- `trackProviderUsage(providerId, tokens, cost)` - Record usage metrics
- `resolveModelFallbacks(modelRef)` - Get fallback model chain

### Provider Integration
- **Authentication**: API key management and validation
- **Rate Limiting**: Request throttling and retry logic
- **Error Handling**: Standardized error response processing
- **Usage Tracking**: Token and cost accounting
- **Feature Detection**: Model capability discovery

## Business Logic

### Provider Registration
1. **Discovery**: Scan configuration for enabled providers
2. **Authentication**: Validate API keys and credentials
3. **Capability Detection**: Query available models and features
4. **Rate Limit Setup**: Configure request throttling parameters
5. **Health Monitoring**: Establish provider health check routines
6. **Model Catalog**: Build unified model catalog across providers

### Request Routing Logic
- **Model Resolution**: Map model references to provider endpoints
- **Load Balancing**: Distribute requests across available providers
- **Fallback Handling**: Automatic failover to backup providers
- **Context Management**: Handle different context window sizes
- **Format Translation**: Convert between provider-specific formats

### Authentication Management
- **API Key Storage**: Secure credential storage and retrieval
- **Token Refresh**: Automatic token renewal for OAuth providers
- **Profile Management**: Multiple credential profiles per provider
- **Environment Variables**: Fallback to environment-based credentials
- **Credential Validation**: Real-time validation of authentication status

### Usage Tracking
- **Token Counting**: Accurate token usage measurement
- **Cost Calculation**: Real-time cost tracking and budgeting
- **Rate Limit Monitoring**: Track API quota consumption
- **Performance Metrics**: Latency and throughput monitoring
- **Error Rate Tracking**: Success/failure ratio analysis

## Data Structures

### Provider Registry
```typescript
interface ProviderRegistry {
  providers: Map<string, Provider>;
  modelCatalog: Map<string, ModelRef>;
  rateLimiters: Map<string, RateLimiter>;
  usageTrackers: Map<string, UsageTracker>;
  healthStatus: Map<string, ProviderHealth>;
}
```

### Provider Instance
```typescript
interface Provider {
  id: string;
  name: string;
  config: ProviderSettings;
  client: APIClient;
  models: ModelDefinition[];
  capabilities: string[];
  lastHealthCheck: number;
  status: "healthy" | "degraded" | "offline";
}
```

### Usage Tracker
```typescript
interface UsageTracker {
  providerId: string;
  totalTokens: number;
  totalRequests: number;
  totalCost: number;
  dailyUsage: DailyUsage[];
  rateLimits: RateLimitStatus;
  errorCount: number;
  averageLatency: number;
}
```

### Rate Limiter
```typescript
interface RateLimiter {
  provider: string;
  requestsPerMinute: number;
  tokensPerMinute: number;
  concurrentRequests: number;
  queue: PendingRequest[];
  lastReset: number;
}
```

## Constraints

### API Limits
- **Rate Limits**: Provider-specific request and token limits
- **Context Windows**: Variable context length limits per model
- **Concurrent Requests**: Maximum simultaneous requests per provider
- **Timeout Limits**: Request timeout constraints (typically 30-300 seconds)
- **Retry Policies**: Exponential backoff with jitter for failed requests

### Security Restrictions
- **API Key Security**: Secure storage and transmission of credentials
- **Network Access**: HTTPS-only communication with providers
- **Data Privacy**: No logging of sensitive conversation content
- **Audit Trail**: Comprehensive logging of API calls and usage
- **Compliance**: GDPR/CCPA compliance for data handling

### Resource Constraints
- **Memory Usage**: Provider client instances and response caching
- **Network Bandwidth**: Large response handling and streaming
- **CPU Usage**: JSON parsing and response processing overhead
- **Storage**: Usage tracking and response caching storage
- **Connection Pools**: HTTP connection management and reuse

## Edge Cases

### Provider Outages
- **Service Unavailable**: Automatic failover to backup providers
- **Partial Outages**: Model-specific availability detection
- **Rate Limit Exceeded**: Queue requests and implement backoff
- **Authentication Failures**: Credential validation and refresh
- **Timeout Handling**: Graceful request timeout and retry

### Model Compatibility Issues
- **Unsupported Features**: Graceful degradation for missing capabilities
- **Format Differences**: Request/response format standardization
- **Context Overflow**: Automatic conversation pruning and summarization
- **Version Mismatches**: Handle model version compatibility
- **Regional Restrictions**: Geographic availability handling

### Configuration Problems
- **Invalid Credentials**: Clear error messages for authentication failures
- **Missing Models**: Fallback to available models
- **Network Connectivity**: Offline mode and connection retry logic
- **Configuration Conflicts**: Validation of provider configuration
- **Legacy Configuration**: Migration of old provider settings

### Performance Issues
- **High Latency**: Request timeout and performance monitoring
- **Memory Leaks**: Proper cleanup of provider client instances
- **Response Size**: Handling of large model responses
- **Concurrent Load**: Connection pooling and request queuing
- **Cache Invalidation**: Model response caching strategies

## Technical Debt

### Current Issues
- **Provider Abstraction**: Inconsistent interfaces between providers
- **Error Handling**: Non-uniform error response formats
- **Configuration Complexity**: Provider-specific configuration variations
- **Authentication Logic**: Different auth patterns across providers
- **Feature Detection**: Manual capability mapping instead of dynamic discovery

### Performance Problems
- **Synchronous Calls**: Blocking API calls impact gateway performance
- **No Connection Pooling**: New connections for each request
- **Response Buffering**: Full response buffering for streaming APIs
- **Cache Misses**: Inefficient caching strategies
- **Retry Logic**: Suboptimal exponential backoff implementation

### Maintenance Challenges
- **Provider Updates**: Manual updates for new models and features
- **Testing**: Limited integration testing with real provider APIs
- **Monitoring**: Insufficient provider health monitoring
- **Documentation**: Outdated provider integration documentation
- **Version Management**: No systematic provider SDK versioning

### Missing Features
- **Streaming Responses**: Partial support for streaming model responses
- **Cost Optimization**: Intelligent model selection based on cost
- **A/B Testing**: No framework for testing different providers
- **Multi-Region**: No geographic distribution of provider requests
- **Batch Processing**: No batch API support for efficiency

## Dependencies

### Core Dependencies
- **axios/fetch**: HTTP client for API requests
- **jsonschema**: Response validation and parsing
- **retry**: Configurable retry logic implementation
- **rate-limiter**: Request rate limiting and queuing
- **crypto**: Secure credential handling and storage

### Provider SDKs
- **OpenAI**: Official OpenAI API client
- **Anthropic**: Claude API integration
- **Google AI**: Gemini model access
- **Azure OpenAI**: Enterprise OpenAI service
- **Cohere**: Cohere model integration

### Internal Dependencies
- **config**: Provider configuration and validation
- **auth**: Authentication and credential management
- **logging**: Request/response logging and monitoring
- **metrics**: Usage tracking and performance monitoring
- **cache**: Response caching and optimization

### Optional Dependencies
- **prometheus**: Metrics collection and monitoring
- **redis**: Distributed caching for provider responses
- **vault**: Enterprise secret management integration
- **opentelemetry**: Distributed tracing for provider calls
- **datadog**: APM monitoring for provider performance

### Platform Dependencies
- **Node.js**: v18+ for modern JavaScript features
- **TLS**: Secure communication with provider APIs
- **DNS**: Domain name resolution for provider endpoints
- **Network**: Reliable internet connectivity
- **Time Sync**: Accurate timestamps for rate limiting and logging