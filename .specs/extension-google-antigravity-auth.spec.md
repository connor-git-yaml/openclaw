# Extension Specification: Google Antigravity Auth

## Intent

Google Antigravity Auth extension provides OAuth2 authentication flow for accessing Google's Antigravity service (Cloud Code Assist). It implements PKCE-based OAuth flow with localhost callback server, enabling users to authenticate with their Google accounts and access premium AI models through Google Cloud infrastructure. The extension handles the complex OAuth dance and credential management required for Google's enterprise AI services.

## Interface

### Plugin Registration
- **ID**: `google-antigravity-auth`
- **Name**: Google Antigravity Auth
- **Type**: Provider Plugin (Authentication)
- **Entry Point**: `index.ts` exports plugin definition
- **API Surface**: Registers OAuth provider with authentication flow

### Authentication Flow
- **OAuth Method**: PKCE (Proof Key for Code Exchange) with S256 challenge
- **Callback Server**: Local HTTP server on port 51121 for OAuth redirects
- **Manual Mode**: Fallback for remote/WSL environments without browser access
- **Token Management**: Access token, refresh token, and expiration handling

### Provider Integration
- **Provider ID**: `google-antigravity`
- **Default Model**: `google-antigravity/claude-opus-4-6-thinking`
- **Cloud Project**: Automatic project ID discovery or fallback default
- **Scopes**: Cloud platform, user info, code completion, experiments

## Business Logic

### OAuth Flow Implementation
1. **PKCE Generation**: Creates code verifier and SHA256 challenge
2. **Auth URL Construction**: Builds Google OAuth URL with required parameters
3. **Callback Handling**: Starts local server to receive OAuth redirect
4. **Code Exchange**: Exchanges authorization code for access/refresh tokens
5. **User Info Retrieval**: Fetches user email and Google Cloud project ID

### Environment Detection
- **WSL Detection**: Identifies WSL/WSL2 environments for manual flow
- **Remote Detection**: Detects remote sessions requiring manual OAuth
- **Browser Integration**: Automatic browser opening for seamless flow
- **Fallback Strategy**: Manual URL copy-paste when automatic flow unavailable

### Token Management
- **Credential Storage**: Secure storage of OAuth tokens and metadata
- **Expiration Handling**: Automatic token refresh with safety margins
- **Profile Creation**: User-specific credential profiles based on email
- **Configuration Updates**: Automatic model configuration and defaults

### Project Discovery
- **Endpoint Probing**: Tests multiple Code Assist endpoints for availability
- **Project Resolution**: Discovers user's Google Cloud project automatically
- **Fallback Project**: Uses default project when discovery fails
- **API Integration**: Proper headers and metadata for Google services

## Data Structures

### OAuth Configuration
```typescript
OAuthConfig {
  CLIENT_ID: string; // Base64 encoded client ID
  CLIENT_SECRET: string; // Base64 encoded client secret
  REDIRECT_URI: "http://localhost:51121/oauth-callback";
  SCOPES: string[]; // Cloud platform and user permissions
}
```

### PKCE Parameters
```typescript
PKCEParams {
  verifier: string; // 32-byte random hex string
  challenge: string; // SHA256 base64url encoded challenge
  state: string; // Random state for CSRF protection
}
```

### Token Response
```typescript
TokenResult {
  access: string; // Access token for API calls
  refresh: string; // Refresh token for renewal
  expires: number; // Expiration timestamp with safety margin
  email?: string; // User email from userinfo
  projectId: string; // Google Cloud project ID
}
```

### Credential Profile
```typescript
CredentialProfile {
  type: "oauth";
  provider: "google-antigravity";
  access: string;
  refresh: string;
  expires: number;
  email?: string;
  projectId: string;
}
```

## Constraints

### Technical Limitations
- **Localhost Binding**: Callback server requires localhost:51121 availability
- **Network Dependencies**: Internet connectivity for OAuth and token exchange
- **Browser Requirements**: Automatic flow needs default browser configuration
- **Platform Restrictions**: Limited functionality in containerized environments

### OAuth Constraints
- **Google Account Required**: Users must have valid Google accounts
- **Cloud Project Access**: Requires Google Cloud project for API access
- **Scope Limitations**: Limited to predefined OAuth scopes
- **Rate Limiting**: Subject to Google's OAuth rate limits

### Environment Limitations
- **WSL/Remote Handling**: Reduced functionality in non-local environments
- **Firewall Issues**: Corporate firewalls may block callback server
- **Port Conflicts**: Port 51121 conflicts require manual fallback
- **Certificate Issues**: SSL/TLS issues in enterprise environments

## Edge Cases

### Authentication Failures
- **Browser Unavailable**: No default browser configured for automatic opening
- **Callback Timeout**: User takes too long to complete OAuth flow
- **State Mismatch**: CSRF protection triggering on legitimate requests
- **Token Exchange Errors**: OAuth code exchange failures from Google

### Network Issues
- **Connectivity Problems**: Internet access issues during authentication
- **Proxy Interference**: Corporate proxies blocking OAuth traffic
- **DNS Resolution**: Localhost resolution problems in some environments
- **Port Binding Failures**: Port 51121 already in use by other applications

### Environment Detection
- **WSL Version Detection**: Difficulty distinguishing between WSL1/WSL2
- **Remote Session Detection**: False positives for remote environment detection
- **Container Environments**: Docker/LXC environments requiring manual flow
- **SSH Forwarding**: Port forwarding issues in SSH sessions

### Project Discovery
- **Multiple Projects**: User has multiple Google Cloud projects available
- **Permission Issues**: Insufficient permissions to list or access projects
- **API Unavailability**: Code Assist endpoints temporarily unavailable
- **Project Migration**: Google Cloud project ID changes or migration

## Technical Debt

### Security Concerns
- **Hardcoded Credentials**: Client ID and secret embedded in code (base64 encoded)
- **Token Storage**: Plain text token storage in configuration files
- **PKCE Implementation**: Manual PKCE implementation vs. library usage
- **State Validation**: Limited state validation beyond simple string comparison

### Error Handling
- **Exception Management**: Broad catch-all error handling masking specific issues
- **User Experience**: Cryptic error messages for authentication failures
- **Retry Logic**: No automatic retry for transient network issues
- **Graceful Degradation**: Limited fallback options for authentication failures

### Code Organization
- **Monolithic Implementation**: All functionality in single large file
- **Hardcoded Values**: URLs, timeouts, and configuration scattered throughout
- **Platform Detection**: Fragile OS and environment detection logic
- **Testing Challenges**: OAuth flow difficult to unit test and mock

### Maintenance Issues
- **Google API Changes**: Vulnerability to Google OAuth API evolution
- **Environment Support**: Adding new environment support requires code changes
- **Configuration Management**: Limited flexibility in OAuth configuration
- **Monitoring and Debugging**: Insufficient logging for troubleshooting auth issues

## Dependencies

### External Dependencies
- **Google OAuth2**: Google's authentication and authorization services
- **Google Cloud APIs**: Code Assist and user info APIs
- **Node.js HTTP**: Built-in HTTP server for OAuth callback handling
- **Crypto Module**: PKCE code challenge generation and validation

### Internal Dependencies
- **openclaw/plugin-sdk**: Core plugin framework and provider registration
- **Authentication System**: Credential storage and profile management
- **Configuration System**: Model configuration and default assignment
- **Network Stack**: HTTP client for OAuth token exchange

### Runtime Requirements
- **Network Connectivity**: Internet access for Google OAuth services
- **Local Server**: Ability to bind HTTP server on localhost:51121
- **File System**: Configuration and credential file storage
- **Browser Access**: Default browser for automatic OAuth flow initiation