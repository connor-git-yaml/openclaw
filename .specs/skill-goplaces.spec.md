# GoPlaces Skill Specification

## Intent

Provide command-line interface to Google Places API (New) for location discovery, place details retrieval, and business information lookup. Enables location-based search, geocoding resolution, and business data access for mapping and local discovery workflows.

## Interface

### Core Operations
- **Text Search**: Find places by name, type, or description
- **Place Resolution**: Convert addresses to structured place data
- **Place Details**: Retrieve comprehensive information about specific places
- **Review Access**: Get user reviews and ratings for businesses

### Command Structure
- `goplaces search "QUERY" [--options]` - Search for places
- `goplaces resolve "ADDRESS" --limit N` - Geocode address to places
- `goplaces details PLACE_ID [--reviews]` - Get detailed place information
- `goplaces search "QUERY" --json` - JSON output for scripting

### Search Modifiers
- `--open-now` - Filter to currently open businesses
- `--min-rating N` - Minimum rating requirement (1-5)
- `--lat LAT --lng LNG --radius-m METERS` - Geographic bias
- `--type TYPE` - Filter by place type (restaurant, store, etc.)
- `--limit N` - Limit number of results
- `--page-token TOKEN` - Pagination for additional results

### Output Formats
- Human-readable default with color formatting
- JSON output (`--json`) for programmatic processing
- Color control (`--no-color` or `NO_COLOR` environment variable)

## Business Logic

### Search and Discovery Engine
1. Execute text-based searches against Google Places API
2. Apply filtering criteria (rating, type, status, location)
3. Handle geographic biasing for local search results
4. Process pagination for comprehensive result sets
5. Parse and format place information for display

### Geocoding and Resolution
1. Convert natural language addresses to structured place data
2. Resolve ambiguous location references to specific places
3. Handle multiple results for ambiguous queries
4. Provide confidence scoring for resolution accuracy

### Place Information Aggregation
1. Retrieve comprehensive place details using place IDs
2. Aggregate business information (hours, contact, website)
3. Collect user reviews and rating data
4. Format structured data for human consumption or API usage

### Geographic Context Management
1. Apply location bias based on user coordinates
2. Handle radius-based search constraints
3. Support distance calculation and ranking
4. Manage geographic boundary and region filtering

## Data Structures

### Place Result
```typescript
interface PlaceResult {
  placeId: string;             // Google Places unique identifier
  name: string;                // business or location name
  address: string;             // formatted address string
  coordinates: {
    latitude: number;
    longitude: number;
  };
  rating?: number;             // user rating (1-5 scale)
  priceLevel?: number;         // price range (0-4, free to very expensive)
  types: string[];             // place types (restaurant, store, etc.)
  isOpenNow?: boolean;         // current operational status
  businessStatus?: string;     // operational status detail
  websiteUrl?: string;         // official website
  phoneNumber?: string;        // contact phone number
  photos: PlacePhoto[];        // associated images
}
```

### Place Details
```typescript
interface PlaceDetails extends PlaceResult {
  description?: string;        // detailed description
  openingHours: OpeningHours;  // detailed hours information
  reviews: PlaceReview[];      // user reviews and ratings
  amenities: string[];         // available features and services
  accessibility: AccessibilityInfo;
  parkingOptions: string[];    // parking availability
  paymentMethods: string[];    // accepted payment types
  serviceOptions: string[];   // delivery, takeout, etc.
}
```

### Place Review
```typescript
interface PlaceReview {
  authorName: string;          // reviewer name
  authorProfilePhoto?: string; // reviewer profile image
  rating: number;              // review rating (1-5)
  text: string;                // review content
  timestamp: Date;             // review publication date
  language: string;            // review language code
}
```

### Search Query
```typescript
interface SearchQuery {
  text: string;                // search query text
  location?: {
    latitude: number;
    longitude: number;
    radius: number;            // search radius in meters
  };
  filters: {
    openNow?: boolean;
    minRating?: number;        // minimum rating filter
    maxPriceLevel?: number;    // maximum price level
    type?: string;             // single place type filter
  };
  pagination: {
    limit: number;             // result count limit
    pageToken?: string;        // next page token
  };
}
```

### API Configuration
```typescript
interface PlacesConfig {
  apiKey: string;              // Google Places API key
  baseUrl?: string;            // API endpoint override
  language?: string;           // preferred response language
  region?: string;             // regional bias
  timeout: number;             // request timeout seconds
  rateLimits: {
    requestsPerSecond: number;
    dailyQuota: number;
  };
}
```

## Constraints

### Google Places API Limitations
- **API Key Required**: Valid Google Cloud Console API key mandatory
- **Usage Quotas**: Daily and per-second request limits
- **Billing Required**: Google Cloud billing account needed for API access
- **Geographic Coverage**: Service availability varies by region

### Search and Query Constraints
- **Single Type Filter**: API accepts only one place type per request
- **Text Length**: Search query text has character length limits
- **Result Limits**: Maximum number of results per request restricted
- **Pagination Tokens**: Page tokens expire after short time periods

### Data and Privacy Restrictions
- **Attribution Requirements**: Google Places data requires proper attribution
- **Caching Limitations**: Restricted caching of place data
- **Commercial Usage**: Additional terms for commercial applications
- **Data Export**: Limitations on bulk data extraction

### Rate Limiting and Performance
- **Request Frequency**: Limits on requests per second per API key
- **Burst Handling**: Temporary rate limit increases not guaranteed
- **Response Time**: API response times vary by query complexity
- **Concurrent Requests**: Limits on simultaneous API calls

## Edge Cases

### API and Authentication Issues
- **Invalid API Key**: Expired, restricted, or malformed API credentials
- **Quota Exceeded**: Daily or per-second limits reached
- **Billing Problems**: Google Cloud billing account suspended
- **Service Outage**: Google Places API temporarily unavailable
- **Regional Blocking**: API access restricted in certain geographic regions

### Query and Search Problems
- **No Results Found**: Search queries returning empty result sets
- **Ambiguous Queries**: Multiple possible interpretations of search text
- **Invalid Coordinates**: Malformed latitude/longitude values
- **Unsupported Place Types**: Using invalid or deprecated place type filters
- **Character Encoding**: Non-ASCII characters in search queries

### Data Quality and Consistency
- **Outdated Information**: Place data not reflecting current business status
- **Missing Details**: Incomplete information for specific places
- **Conflicting Data**: Inconsistencies between basic and detailed place information
- **Language Mismatches**: Requested language not available for place data
- **Photo Unavailability**: Image references returning broken links

### Network and Performance Issues
- **Connection Timeouts**: Slow network causing API request failures
- **Partial Responses**: Incomplete data due to network interruption
- **Retry Logic**: Handling transient failures and retry strategies
- **Large Result Sets**: Performance issues with high-limit queries
- **Concurrent Usage**: Resource contention with other API users

## Technical Debt

### Error Handling and Recovery
- Basic error messages without specific guidance for Google API errors
- No automatic retry logic for transient failures
- Limited context for quota and billing-related errors
- Poor handling of partial or malformed API responses

### Configuration and Setup
- API key management basic and potentially insecure
- No validation of API key permissions and quotas
- Limited guidance for Google Cloud Console setup
- Missing configuration validation and testing tools

### Performance and Optimization
- No local caching of frequently accessed place data
- Sequential API requests instead of batching where possible
- No optimization for repeated searches in same geographic area
- Limited request deduplication for similar queries

### Feature Coverage Gaps
- No support for advanced Google Places features (photos API, autocomplete)
- Missing integration with Google Maps for visual representation
- Limited support for place management operations (add, edit)
- No bulk operations for processing multiple locations

## Dependencies

### Core Dependencies
- **goplaces Binary**: Primary CLI tool for Google Places API access
- **HTTP Client**: For Google Places API communication
- **JSON Processing**: API request/response serialization

### Google API Dependencies
- **Google Places API (New)**: Modern version of Google Places services
- **Google Cloud Console**: API key management and billing
- **Google Maps Platform**: Underlying mapping and location services

### Authentication Dependencies
- **Google Places API Key**: Required environment variable
- **Google Cloud Billing**: Active billing account for API usage
- **API Quotas**: Configured usage limits and permissions

### System Dependencies
- **Network Connectivity**: HTTPS access to Google APIs
- **Environment Variables**: API key and configuration storage
- **Terminal Support**: Color output and text formatting
- **JSON Processing**: Response parsing and formatting

### Optional Dependencies
- **Homebrew**: Package manager for macOS installation
- **Proxy Support**: HTTP proxy configuration for corporate networks
- **Output Formatting**: Advanced text processing and display tools
- **Geographic Libraries**: Enhanced coordinate and distance calculations

### Development Dependencies
- **Go Toolchain**: For building goplaces from source
- **Testing Framework**: Automated testing of Google Places integration
- **API Documentation**: Google Places API reference materials
- **Build System**: Cross-platform compilation and distribution