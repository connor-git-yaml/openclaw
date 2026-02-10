# Extension Specification: Copilot Proxy

## Intent

Copilot Proxy extension enables OpenClaw to utilize GitHub Copilot's language models through a local proxy server. It integrates with VS Code's Copilot Proxy extension to provide access to premium models (GPT-5.x, Claude-4.x, Gemini-3, etc.) via OpenAI-compatible API endpoints. This allows users to leverage their existing Copilot subscriptions for advanced AI model access within the OpenClaw ecosystem.

## Interface

### Plugin Registration
- **ID**: `copilot-proxy`
- **Name**: Copilot Proxy
- **Type**: Provider Plugin (LLM Models)
- **Entry Point**: `index.ts` exports plugin definition
- **API Surface**: Registers model provider with authentication flow

### Provider Configuration
- **Base URL**: Local proxy endpoint (default: http://localhost:3000/v1)
- **API Key**: Static value ("n/a" - auth handled by proxy)
- **Model Selection**: Dynamic list of supported models
- **Auth Method**: Custom authentication flow with URL/model configuration

### Supported Models
- **GPT Series**: gpt-5.2, gpt-5.2-codex, gpt-5.1, gpt-5.1-codex, gpt-5.1-codex-max, gpt-5-mini
- **Claude Series**: claude-opus-4.6, claude-opus-4.5, claude-sonnet-4.5, claude-haiku-4.5
- **Gemini Series**: gemini-3-pro, gemini-3-flash
- **Grok Series**: grok-code-fast-1

## Business Logic

### Provider Registration
- Registers as "copilot-proxy" provider with OpenClaw model system
- Configures OpenAI-compatible API interface for model communication
- Sets up authentication workflow for local proxy server connection
- Establishes default model mappings for agent configuration

### Authentication Flow
1. **Proxy Configuration**: User provides base URL for local Copilot Proxy server
2. **Model Selection**: User selects available models from predefined list
3. **Profile Creation**: Creates "copilot-proxy:local" credential profile
4. **Config Patching**: Updates OpenClaw configuration with provider settings
5. **Default Assignment**: Sets first selected model as system default

### Request Processing
- **API Compatibility**: Uses OpenAI completions format for all models
- **Header Management**: Disables authentication header (proxy handles auth)
- **Model Routing**: Routes requests to appropriate Copilot models via proxy
- **Response Handling**: Processes standard OpenAI completion responses

### Configuration Management
- **URL Normalization**: Ensures proper /v1 endpoint suffix
- **Model Parsing**: Handles comma-separated model ID input
- **Deduplication**: Removes duplicate model entries from configuration
- **Validation**: Validates URLs and ensures at least one model is selected

## Data Structures

### Model Definition
```typescript
ModelDefinition {
  id: string;
  name: string;
  api: "openai-completions";
  reasoning: false;
  input: ["text", "image"];
  cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 };
  contextWindow: 128000;
  maxTokens: 8192;
}
```

### Provider Configuration
```typescript
ProviderConfig {
  baseUrl: string;
  apiKey: "n/a";
  api: "openai-completions";
  authHeader: false;
  models: ModelDefinition[];
}
```

### Authentication Profile
```typescript
AuthProfile {
  profileId: "copilot-proxy:local";
  credential: {
    type: "token";
    provider: "copilot-proxy";
    token: "n/a";
  };
}
```

## Constraints

### Technical Limitations
- **Local Dependency**: Requires VS Code Copilot Proxy extension to be running
- **Port Binding**: Default port 3000 may conflict with other local services
- **Model Availability**: Limited to models supported by user's Copilot subscription
- **OpenAI Compatibility**: Restricted to OpenAI completion format (no streaming variants)

### Subscription Requirements
- **GitHub Copilot**: Requires active Copilot subscription for model access
- **Plan Limitations**: Model availability depends on subscription tier
- **Usage Quotas**: Subject to Copilot's usage limits and rate restrictions
- **VS Code Integration**: Requires VS Code with Copilot extension installed

### Network Constraints
- **Localhost Only**: Designed for local proxy connections only
- **No Authentication**: Relies on proxy server for authentication handling
- **Single Instance**: One proxy configuration per OpenClaw installation

## Edge Cases

### Proxy Server Issues
- **Server Unavailable**: Connection failures when proxy is not running
- **Port Changes**: Proxy server running on different port than configured
- **Extension Disabled**: VS Code Copilot extension not active or authenticated
- **Model Unavailability**: Requested models not available in user's subscription

### Configuration Errors
- **Invalid URLs**: Malformed base URL input from user
- **Empty Model Lists**: User accidentally removes all model selections
- **Duplicate Models**: Redundant model entries in configuration
- **Path Issues**: Missing or incorrect /v1 API path suffix

### Authentication Problems
- **Copilot Logout**: VS Code Copilot authentication expires
- **Subscription Expired**: GitHub Copilot subscription becomes inactive
- **Proxy Authentication**: Proxy server fails to authenticate with Copilot service
- **Token Issues**: Proxy token refresh failures

### Runtime Failures
- **Model Switching**: Failures when switching between different model types
- **Context Overflow**: Requests exceeding model context windows
- **Rate Limiting**: Copilot service rate limit enforcement
- **Response Parsing**: Malformed responses from proxy server

## Technical Debt

### Configuration Management
- Hardcoded model list may become outdated as Copilot adds new models
- No automatic discovery of available models from proxy server
- Static context window and token limits don't reflect actual model capabilities
- Missing validation for model compatibility with user's subscription

### Error Handling
- Limited error reporting for proxy connection failures
- No retry mechanisms for transient proxy server issues
- Missing graceful degradation when preferred models unavailable
- Insufficient logging for debugging configuration problems

### Code Organization
- All functionality in single file makes testing and maintenance difficult
- URL normalization logic could be extracted to utility functions
- Model definition generation is repetitive and could be standardized
- Configuration patching logic is complex and hard to follow

## Dependencies

### External Dependencies
- **VS Code**: Required for Copilot Proxy extension hosting
- **GitHub Copilot**: Core service providing model access
- **Copilot Proxy Extension**: VS Code extension serving local API
- **Network Stack**: HTTP client for local proxy communication

### Internal Dependencies
- **openclaw/plugin-sdk**: Core plugin framework and provider registration
- **Model System**: Integration with OpenClaw's model management
- **Configuration System**: Provider and credential configuration
- **Authentication Framework**: Profile and credential management

### Runtime Requirements
- **VS Code Running**: Active VS Code instance with Copilot extension
- **Proxy Server Active**: Copilot Proxy extension serving on configured port
- **GitHub Authentication**: Valid GitHub account with Copilot access
- **Network Connectivity**: Local HTTP communication and external Copilot API access