# Extension Specification: Google Gemini CLI Auth

## Intent

Google Gemini CLI Auth extension provides OAuth2 authentication for Google's Gemini CLI service, enabling access to Gemini models through Google Cloud Code Assist. It implements PKCE-based OAuth flow with automatic client credential detection from environment variables or installed Gemini CLI tools, supporting both free and paid tiers of Google's AI services through standardized authentication patterns.

## Interface

### Plugin Registration
- **ID**: `google-gemini-cli-auth`
- **Name**: Google Gemini CLI Auth
- **Type**: Provider Plugin (Authentication)
- **Entry Point**: `index.ts` exports plugin definition
- **API Surface**: Registers OAuth provider with credential auto-detection

### Authentication Methods
- **OAuth Flow**: PKCE-based authentication with localhost callback
- **Client Discovery**: Automatic detection of OAuth credentials from multiple sources
- **Environment Variables**: Support for custom OAuth client configurations
- **Gemini CLI Integration**: Leverages existing Gemini CLI installations

### Environment Variable Support
```
OPENCLAW_GEMINI_OAUTH_CLIENT_ID
OPENCLAW_GEMINI_OAUTH_CLIENT_SECRET
GEMINI_CLI_OAUTH_CLIENT_ID
GEMINI_CLI_OAUTH_CLIENT_SECRET
```

### Provider Configuration
- **Provider ID**: `google-gemini-cli`
- **Default Model**: `google-gemini-cli/gemini-3-pro-preview`
- **Service Tiers**: Free, legacy, and standard tier support
- **Callback Port**: localhost:8085 for OAuth redirects

## Business Logic

### Client Credential Resolution
1. **Environment Variables**: Checks predefined environment variable names
2. **File System Search**: Scans system for installed Gemini CLI credentials
3. **Path Discovery**: Searches PATH for Gemini CLI executables
4. **Configuration Extraction**: Parses found configuration files for OAuth clients
5. **Fallback Strategies**: Multiple credential sources with priority ordering

### OAuth Implementation
- **PKCE Flow**: Secure OAuth with code challenge/verifier pairs
- **State Management**: CSRF protection with random state tokens
- **Token Exchange**: Authorization code to access/refresh token exchange
- **User Info Retrieval**: Email and profile information fetching
- **Tier Detection**: Service tier identification for quota management

### Credential Management
- **Profile Creation**: User-specific profiles based on email addresses
- **Token Storage**: Secure storage of OAuth tokens with expiration
- **Refresh Logic**: Automatic token renewal before expiration
- **Multi-Account**: Support for multiple Google accounts simultaneously

### Service Integration
- **Cloud Platform APIs**: Integration with Google Cloud services
- **Code Assist**: Google's AI-powered code completion and assistance
- **Quota Management**: Proper handling of service tier limitations
- **Model Access**: Dynamic model availability based on tier and permissions

## Data Structures

### OAuth Credentials
```typescript
GeminiCliOAuthCredentials {
  access: string; // Bearer token for API calls
  refresh: string; // Refresh token for renewal
  expires: number; // Expiration timestamp
  email?: string; // User email address
  tier?: string; // Service tier (free/legacy/standard)
}
```

### Client Configuration
```typescript
OAuthClientConfig {
  clientId: string; // OAuth client ID
  clientSecret: string; // OAuth client secret
  redirectUri: "http://localhost:8085/oauth2callback";
  scopes: string[]; // Required OAuth scopes
}
```

### PKCE Parameters
```typescript
PKCEFlow {
  verifier: string; // Code verifier for PKCE
  challenge: string; // Code challenge (SHA256)
  state: string; // CSRF protection state
  authUrl: string; // Complete authorization URL
}
```

### Provider Profile
```typescript
ProviderProfile {
  profileId: string; // "google-gemini-cli:{email}"
  credential: GeminiCliOAuthCredentials;
  tier?: string; // Service tier information
  quotaInfo?: QuotaDetails;
}
```

## Constraints

### Authentication Requirements
- **Google Account**: Valid Google account for OAuth authentication
- **OAuth Client**: Configured OAuth client credentials (environment or CLI)
- **Network Access**: Internet connectivity for OAuth and API calls
- **Browser Access**: Default browser for interactive OAuth flow

### Technical Limitations
- **Port Requirements**: localhost:8085 must be available for OAuth callback
- **Environment Detection**: Limited to specific environment variable patterns
- **CLI Dependencies**: Gemini CLI auto-discovery requires specific installation patterns
- **Tier Restrictions**: Model access limited by Google service tier

### Platform Constraints
- **File System Access**: Required for CLI credential discovery
- **Path Scanning**: PATH environment variable access for executable discovery
- **Configuration Parsing**: JSON parsing of various Gemini CLI config formats
- **Permission Requirements**: Read access to system and user directories

## Edge Cases

### Credential Discovery Failures
- **Missing Credentials**: No OAuth client found in any supported location
- **Invalid Configuration**: Malformed JSON or missing required fields
- **Permission Denied**: Insufficient file system permissions for credential access
- **Multiple Installations**: Conflicting Gemini CLI installations with different credentials

### OAuth Flow Issues
- **Callback Failures**: OAuth callback server binding or request processing failures
- **Browser Problems**: No default browser or browser launch failures
- **Token Exchange Errors**: Google OAuth service rejecting authorization codes
- **Network Interruptions**: Connectivity issues during OAuth flow

### Service Tier Complications
- **Quota Exhaustion**: Service tier quotas exceeded during usage
- **Tier Changes**: User's service tier changing during session
- **Model Unavailability**: Requested models not available for user's tier
- **Permission Escalation**: Insufficient permissions for advanced features

### Multi-Account Scenarios
- **Account Switching**: Multiple Google accounts causing confusion
- **Profile Conflicts**: Email address collisions between different OAuth clients
- **Credential Mixing**: Different OAuth clients for same user account
- **Session Management**: Concurrent sessions with different accounts

## Technical Debt

### Security Considerations
- **Credential Storage**: OAuth client secrets stored in environment variables
- **Token Security**: Access tokens in memory and configuration files
- **File System Search**: Scanning system for credentials could expose sensitive data
- **Network Security**: HTTP callback server for OAuth (localhost only)

### Code Organization
- **Monolithic OAuth Implementation**: Large oauth.ts file combining multiple concerns
- **Hardcoded Values**: URLs, ports, and timeouts scattered throughout code
- **Error Handling**: Inconsistent error handling patterns across OAuth flow
- **Testing Limitations**: OAuth flow difficult to unit test and mock

### Configuration Management
- **Environment Variable Proliferation**: Multiple environment variables for similar purposes
- **CLI Discovery Logic**: Complex file system scanning with brittle path assumptions
- **Configuration Validation**: Limited validation of discovered OAuth credentials
- **Fallback Strategy**: Unclear prioritization of credential sources

### Performance Issues
- **File System Scanning**: Potentially expensive directory traversal operations
- **Synchronous Operations**: Blocking file system operations during startup
- **Network Timeouts**: Fixed timeouts may not suit all network conditions
- **Resource Cleanup**: Incomplete cleanup of temporary OAuth resources

## Dependencies

### External Dependencies
- **Google OAuth2**: Google's authentication and authorization infrastructure
- **Google Cloud APIs**: Code Assist and user information services
- **Node.js HTTP**: Built-in HTTP server for OAuth callback handling
- **File System APIs**: Node.js fs module for credential discovery

### Internal Dependencies
- **openclaw/plugin-sdk**: Core plugin framework and provider registration
- **Authentication System**: Profile and credential management
- **Configuration System**: Model configuration and environment variable handling
- **Network Stack**: HTTP client for OAuth and API communication

### Runtime Requirements
- **Network Connectivity**: Internet access for Google services and OAuth flow
- **File System Access**: Read permissions for credential discovery and configuration
- **Environment Variables**: Access to process environment for credential configuration
- **Browser Integration**: Default browser for interactive OAuth authentication