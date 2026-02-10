# Food Order Skill Specification

## Intent

Enable safe and controlled food ordering through Foodora delivery service via the ordercli CLI tool. Provides order history access, reordering capabilities, and order tracking with explicit user confirmation requirements for financial transactions.

## Interface

### Core Operations
- **History Access**: View previous Foodora orders and details
- **Reorder Preview**: Show what would be reordered without placing order
- **Order Placement**: Reorder previous orders with explicit user confirmation
- **Order Tracking**: Monitor active orders and ETAs
- **Account Management**: Login, country configuration, and session management

### Safety-First Command Structure
- `ordercli foodora history --limit N` - Browse order history
- `ordercli foodora history show ORDER_CODE` - View order details
- `ordercli foodora reorder ORDER_CODE` - **Preview only** (no charges)
- `ordercli foodora reorder ORDER_CODE --confirm` - **Place order** (requires explicit approval)
- `ordercli foodora orders [--watch]` - Track active orders

### Authentication Methods
- **Password Login**: `ordercli foodora login --email EMAIL --password-stdin`
- **Browser Session**: `ordercli foodora session chrome --url URL --profile PROFILE` (preferred)
- **Configuration**: Per-country setup via `ordercli foodora config set --country CODE`

## Business Logic

### Safety-First Order Flow
1. **Discovery**: User expresses food ordering intent
2. **History Review**: Show recent order history for selection
3. **Preview Generation**: Display exact order contents and cost without confirmation
4. **Explicit Confirmation**: Require clear user approval ("yes", "confirm", "place the order")
5. **Order Execution**: Only then execute reorder with `--confirm` flag
6. **Order Tracking**: Provide ongoing status updates and ETA

### Authentication and Session Management
1. Support multiple authentication methods (password, browser session)
2. Maintain persistent login state across operations
3. Handle country-specific Foodora configurations
4. Manage temporary configurations for testing/debugging

### Order History and Analysis
1. Retrieve and display recent order history with pagination
2. Parse order details including items, costs, restaurants, and delivery addresses
3. Present order information in user-friendly format
4. Support machine-readable JSON output for automation

### Risk Mitigation and Validation
1. Never execute confirmed orders without explicit user approval
2. Validate delivery address selections for multi-address accounts
3. Preview all order details before confirmation
4. Provide clear cost breakdown and restaurant information

## Data Structures

### Food Order
```typescript
interface FoodOrder {
  orderCode: string;           // unique order identifier
  restaurant: Restaurant;      // restaurant information
  items: OrderItem[];          // ordered food items
  totalCost: number;           // total order cost
  deliveryAddress: Address;    // delivery location
  orderDate: Date;             // original order timestamp
  status: OrderStatus;         // current order state
  estimatedDelivery?: Date;    // expected delivery time
}
```

### Order Item
```typescript
interface OrderItem {
  name: string;                // item name
  quantity: number;            // number of items
  price: number;               // unit price
  totalPrice: number;          // quantity × price
  customizations?: string[];   // special requests/modifications
  category: string;            // food category
}
```

### Restaurant Info
```typescript
interface Restaurant {
  id: string;                  // restaurant identifier
  name: string;                // restaurant name
  cuisine: string;             // cuisine type
  rating: number;              // customer rating
  deliveryTime: number;        // estimated delivery time (minutes)
  minimumOrder?: number;       // minimum order amount
  deliveryFee: number;         // delivery charge
  isOpen: boolean;             // current availability status
}
```

### Delivery Address
```typescript
interface Address {
  id: string;                  // address identifier
  name?: string;               // address nickname
  street: string;              // street address
  city: string;                // city
  postalCode: string;          // postal/ZIP code
  country: string;             // country code
  instructions?: string;       // delivery instructions
  isDefault: boolean;          // default address flag
}
```

### Order Status
```typescript
interface OrderStatus {
  state: 'pending' | 'confirmed' | 'preparing' | 'in_transit' | 'delivered' | 'cancelled';
  timestamp: Date;             // status timestamp
  estimatedDelivery?: Date;    // updated ETA
  trackingInfo?: string;       // delivery tracking details
  courierInfo?: CourierInfo;   // delivery person information
}
```

## Constraints

### Financial Safety Requirements
- **No Automatic Orders**: Never place orders without explicit user confirmation
- **Preview Mandatory**: Must show order preview before any financial commitment
- **Clear Confirmation**: Require unambiguous approval language
- **No Silent Retries**: Failed orders must not retry automatically

### Platform Dependencies
- **Foodora Account**: Valid customer account with payment method
- **Country Configuration**: Service availability varies by region
- **Browser Integration**: Chrome session auth requires browser access
- **Internet Connectivity**: All operations require stable internet connection

### Authentication Constraints
- **Session Persistence**: Login sessions have expiration times
- **Multi-Factor Auth**: Some accounts may require additional verification
- **Rate Limiting**: API calls subject to Foodora's usage limits
- **Geographic Restrictions**: Service availability limited by location

## Edge Cases

### Authentication and Session Issues
- **Session Expiry**: Login tokens expiring during order process
- **Browser Profile Missing**: Chrome session auth failing due to missing profiles
- **Multi-Factor Required**: Additional authentication steps blocking automation
- **Country Mismatch**: Account country not matching configured country
- **Password Changes**: Account credentials updated externally

### Order and Restaurant Problems
- **Restaurant Closed**: Selected restaurant unavailable during reorder attempt
- **Menu Changes**: Previous order items no longer available
- **Price Updates**: Order costs different from historical pricing
- **Delivery Area**: Address no longer in restaurant's delivery zone
- **Minimum Order**: Historical order below current minimum order requirement

### Payment and Financial Issues
- **Payment Method Expired**: Credit card or payment method no longer valid
- **Insufficient Funds**: Payment declining due to insufficient balance
- **Currency Changes**: Price differences due to currency fluctuations
- **Promotion Expiry**: Previous discounts or promotions no longer valid
- **Tax Updates**: Changed tax rates affecting order total

### Technical and Network Failures
- **API Outages**: Foodora service temporarily unavailable
- **Network Interruption**: Connection lost during order placement
- **Order Duplication**: Technical issues causing multiple order submissions
- **Status Update Delays**: Delayed or missing order status notifications
- **Browser Automation Failures**: Chrome session automation breaking

## Technical Debt

### Safety Mechanism Limitations
- Confirmation detection relies on simple keyword matching
- No automated rollback for accidentally confirmed orders
- Limited validation of user intent before order placement
- Basic error handling for edge cases during order flow

### Authentication Complexity
- Multiple authentication methods create configuration confusion
- No unified session management across authentication types
- Browser session dependency creates fragility
- Limited error context for authentication failures

### Order Management Gaps
- No local caching of order history for offline access
- Limited search and filtering capabilities for order history
- No automated favorite order detection and shortcuts
- Basic order modification support (address changes, cancellations)

### Integration Limitations
- Single platform support (Foodora only)
- No integration with dietary preferences or restrictions
- Limited restaurant recommendation based on order history
- No coordination with calendar or meal planning systems

## Dependencies

### Core Dependencies
- **ordercli Binary**: Primary CLI tool for Foodora integration
- **Go Runtime**: Required for ordercli binary execution
- **HTTP Client**: For Foodora API communication

### Authentication Dependencies
- **Foodora Account**: Valid customer account with active payment method
- **Browser Integration**: Chrome browser for session-based authentication
- **Configuration System**: File-based configuration storage
- **Session Management**: Token persistence and refresh handling

### Network Dependencies
- **HTTPS Connectivity**: Secure communication with Foodora API
- **Geographic Location**: IP-based location verification
- **DNS Resolution**: Domain name resolution for API endpoints
- **Certificate Validation**: SSL/TLS certificate verification

### Browser Dependencies (for session auth)
- **Chrome Browser**: Specific browser for session extraction
- **Browser Profiles**: Access to Chrome user profiles
- **Browser Automation**: Ability to extract session cookies
- **Profile Permissions**: File system access to browser data

### System Dependencies
- **File System**: Configuration and session data storage
- **Process Management**: Command execution and session handling
- **JSON Processing**: Order data parsing and formatting
- **Time/Date**: Order timing and ETA calculations

### Optional Dependencies
- **Notification System**: Order status alert delivery
- **Calendar Integration**: Meal planning and scheduling
- **Expense Tracking**: Financial record keeping
- **Dietary Apps**: Integration with nutrition and dietary management