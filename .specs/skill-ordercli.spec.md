# OrderCLI Skill Specification

## Intent

Provide command-line interface for food delivery service management, primarily focusing on Foodora order tracking, history retrieval, and reordering capabilities. Enables programmatic access to delivery service accounts with order monitoring and automated reordering workflows.

## Interface

### Core Operations
- **Order Tracking**: Monitor active orders with real-time status updates
- **Order History**: Access past orders with detailed information
- **Reordering**: Recreate previous orders with cart management
- **Authentication**: Multiple login methods including browser automation
- **Configuration**: Country/market setup and account management

### Provider Support
- **Foodora**: Full feature support with order management and reordering
- **Deliveroo**: Work-in-progress support with basic history access
- **Multi-Country**: Regional configuration for different markets

### Command Structure
- `ordercli foodora countries` - List supported countries
- `ordercli foodora login --email EMAIL --password-stdin` - Authentication
- `ordercli foodora orders [--watch]` - Active order monitoring
- `ordercli foodora history --limit N` - Order history retrieval
- `ordercli foodora reorder ORDER_CODE [--confirm]` - Order recreation

### Authentication Methods
- **Password Login**: Standard email/password authentication
- **Browser Automation**: Bypass bot protection with real browser
- **Session Import**: Import existing browser sessions without password
- **Cookie Management**: Chrome cookie extraction and reuse

## Business Logic

### Authentication and Session Management
1. Support multiple authentication methods for different scenarios
2. Handle Cloudflare and bot protection with browser automation
3. Manage session persistence and refresh token handling
4. Store credentials securely with configurable storage options

### Order Tracking and Monitoring
1. Retrieve active order status with delivery tracking information
2. Provide real-time updates with watch mode for continuous monitoring
3. Parse order details including restaurant, items, and delivery estimates
4. Handle multiple concurrent orders and status changes

### Historical Order Management
1. Access comprehensive order history with pagination support
2. Extract detailed order information including items, costs, and timing
3. Support order search and filtering capabilities
4. Provide structured data output for integration and analysis

### Reordering Workflow
1. Preview reorder contents before cart modification
2. Handle address selection for multi-address accounts
3. Apply confirmation safeguards to prevent accidental orders
4. Manage cart state and checkout process coordination

### Anti-Bot and Protection Handling
1. Detect and handle Cloudflare protection mechanisms
2. Use browser automation to bypass bot detection
3. Manage browser profiles and session persistence
4. Implement fallback authentication methods

## Data Structures

### Food Order
```typescript
interface FoodOrder {
  orderCode: string;           // unique order identifier
  status: OrderStatus;         // current order state
  restaurant: RestaurantInfo;  // restaurant details
  items: OrderItem[];          // ordered items
  totalCost: number;           // order total amount
  currency: string;            // currency code
  orderTime: Date;             // order placement timestamp
  estimatedDelivery?: Date;    // expected delivery time
  actualDelivery?: Date;       // actual delivery timestamp
  deliveryAddress: Address;    // delivery location
  paymentMethod: string;       // payment method used
}
```

### Order Status
```typescript
interface OrderStatus {
  stage: 'placed' | 'confirmed' | 'preparing' | 'picked_up' | 'in_transit' | 'delivered' | 'cancelled';
  timestamp: Date;             // status timestamp
  description: string;         // human-readable status
  estimatedTime?: number;      // estimated minutes until next stage
  trackingInfo?: TrackingInfo; // delivery tracking details
}

interface TrackingInfo {
  courierName?: string;        // delivery person name
  courierPhone?: string;       // delivery contact
  vehicleType?: string;        // delivery vehicle type
  currentLocation?: Location;  // real-time location
  estimatedArrival: Date;      // updated delivery estimate
}
```

### Restaurant Information
```typescript
interface RestaurantInfo {
  id: string;                  // restaurant identifier
  name: string;                // restaurant name
  cuisine: string;             // cuisine type
  address: string;             // restaurant address
  phone?: string;              // restaurant contact
  rating?: number;             // customer rating
  deliveryFee: number;         // delivery cost
  minimumOrder?: number;       // minimum order amount
  isOpen: boolean;             // current availability
}
```

### Order Item
```typescript
interface OrderItem {
  id: string;                  // item identifier
  name: string;                // item name
  description?: string;        // item description
  quantity: number;            // number ordered
  unitPrice: number;           // price per unit
  totalPrice: number;          // total item cost
  customizations: string[];    // special instructions
  category: string;            // item category
}
```

### User Configuration
```typescript
interface OrderCLIConfig {
  provider: 'foodora' | 'deliveroo';
  country: string;             // country/market code
  credentials: AuthCredentials;
  defaultAddress?: Address;    // default delivery address
  browserProfile?: string;     // browser profile path
  preferences: UserPreferences;
}

interface AuthCredentials {
  email?: string;              // login email
  sessionToken?: string;       // session authentication
  refreshToken?: string;       // token refresh credential
  browserCookies?: string;     // imported browser cookies
  bearerToken?: string;        // API bearer token (Deliveroo)
}
```

### Delivery Address
```typescript
interface Address {
  id?: string;                 // address identifier
  name?: string;               // address nickname
  street: string;              // street address
  city: string;                // city name
  postalCode: string;          // postal code
  country: string;             // country code
  instructions?: string;       // delivery instructions
  isDefault: boolean;          // default address flag
}
```

## Constraints

### Provider-Specific Limitations
- **Foodora**: Full feature support with comprehensive API access
- **Deliveroo**: Limited work-in-progress support with basic functionality
- **Geographic Coverage**: Limited to supported countries/markets per provider
- **Feature Parity**: Different capabilities across food delivery platforms

### Authentication and Security Constraints
- **Bot Protection**: Cloudflare and anti-bot measures requiring browser automation
- **Session Expiry**: Authentication tokens with limited lifetime requiring refresh
- **Account Restrictions**: Provider-imposed limits on API access and automation
- **Browser Dependencies**: Browser automation requiring GUI environment

### Order Management Limitations
- **Reorder Constraints**: Previous orders may no longer be available
- **Restaurant Availability**: Closed restaurants preventing reorder completion
- **Menu Changes**: Item modifications affecting reorder accuracy
- **Address Validation**: Delivery area restrictions for different addresses

### Technical Dependencies
- **Network Connectivity**: Internet access for provider API communication
- **Browser Environment**: GUI environment for browser automation features
- **Configuration Storage**: File system access for credential and setting storage
- **Process Management**: Background processes for order monitoring

## Edge Cases

### Authentication and Session Issues
- **Account Lockout**: Too many failed authentication attempts
- **Session Invalidation**: Tokens becoming invalid during operations
- **Password Changes**: Account credentials updated externally
- **Two-Factor Authentication**: Additional authentication steps blocking automation
- **Account Suspension**: Provider suspending account access

### Order and Restaurant Problems
- **Restaurant Closure**: Sudden closure affecting active orders or reorders
- **Menu Unavailability**: Items no longer available during reorder attempt
- **Delivery Area Changes**: Address no longer in delivery zone
- **Payment Failures**: Credit card issues preventing order completion
- **Order Cancellation**: Unexpected order cancellation by restaurant or provider

### Network and Service Issues
- **Provider API Outage**: Food delivery service temporarily unavailable
- **Rate Limiting**: Too many requests causing temporary blocks
- **Network Timeout**: Slow connections causing operation failures
- **Bot Detection**: Anti-automation measures blocking legitimate use
- **Service Degradation**: Provider service performance issues affecting operations

### Configuration and Environment Problems
- **Configuration Corruption**: Config files becoming invalid or corrupted
- **Browser Profile Issues**: Browser automation setup problems
- **Permission Denied**: File system access issues for configuration storage
- **Multiple Instances**: Concurrent ordercli instances causing conflicts
- **Country/Market Mismatch**: Configuration not matching actual location

## Technical Debt

### Authentication Complexity
- Multiple authentication methods creating configuration complexity
- Browser automation dependency reducing reliability and portability
- Basic credential storage without secure keyring integration
- Limited error recovery for authentication failures

### Provider Integration Gaps
- Foodora-focused development with limited multi-provider support
- Inconsistent API coverage across different food delivery platforms
- Basic error handling for provider-specific failures
- No unified interface for cross-provider operations

### Order Management Limitations
- Basic reorder functionality without intelligent item substitution
- Limited order customization during reorder process
- No batch operations for multiple orders
- Basic order filtering and search capabilities

### User Experience and Automation
- Command-line only interface without graphical components
- Limited progress feedback for long-running operations
- Basic configuration management without setup wizards
- No integration with calendar or scheduling systems

## Dependencies

### Core Dependencies
- **ordercli Binary**: Primary CLI tool for food delivery service integration
- **HTTP Client**: Network communication with provider APIs
- **JSON Processing**: API request/response serialization and parsing
- **Configuration Management**: File-based configuration storage

### Authentication Dependencies
- **Browser Automation**: Puppeteer or similar for bot protection bypass
- **Cookie Management**: Browser cookie extraction and session import
- **Session Storage**: Secure credential and token persistence
- **Cryptographic Libraries**: Token validation and secure storage

### Provider Dependencies
- **Foodora API**: Primary food delivery service integration
- **Deliveroo API**: Secondary service (work-in-progress)
- **Geographic Services**: Country/market configuration and validation
- **Payment Processing**: Integration with provider payment systems

### System Dependencies
- **File System Access**: Configuration and credential storage
- **Network Stack**: HTTPS connectivity for API communication
- **Process Management**: Background monitoring and session management
- **Browser Environment**: GUI environment for browser automation (optional)

### Optional Dependencies
- **Homebrew**: Package manager for macOS installation
- **Go Toolchain**: Alternative installation method from source
- **Notification System**: Order status alerts and notifications
- **Calendar Integration**: Delivery scheduling and time management

### Development Dependencies
- **Testing Framework**: API integration testing and validation
- **Mock Services**: Provider API simulation for development
- **Documentation Tools**: Usage guides and API documentation
- **Build System**: Cross-platform compilation and distribution