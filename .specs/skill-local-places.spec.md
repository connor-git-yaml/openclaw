# Local Places Skill Specification

## Intent

Provide local HTTP proxy server for Google Places API access with enhanced location resolution and place search capabilities. Enables fast local place discovery with intelligent location bias, filtering, and pagination for location-based applications.

## Interface

### Core Operations
- **Location Resolution**: Convert text descriptions to geographic coordinates
- **Place Search**: Find businesses and points of interest with advanced filtering
- **Place Details**: Retrieve comprehensive information for specific places
- **Health Monitoring**: Server status and API connectivity verification

### API Endpoints
- `POST /locations/resolve` - Convert location text to coordinates
- `POST /places/search` - Search places with location bias and filters
- `GET /places/{place_id}` - Get detailed place information
- `GET /ping` - Health check endpoint

### Search Workflow
1. **Location Resolution**: User provides vague location → resolve to coordinates
2. **Preference Collection**: Gather search criteria (type, rating, price, status)
3. **Biased Search**: Execute search with location bias and filters
4. **Result Presentation**: Display results with key information
5. **Detail Retrieval**: Fetch comprehensive place details on request

### Conversation Integration
- Natural language location interpretation
- Interactive preference gathering with numbered choices
- Results presentation with actionable options
- Iterative search refinement based on user feedback

## Business Logic

### Location Resolution Engine
1. Parse natural language location descriptions
2. Geocode addresses and landmark references to precise coordinates
3. Handle ambiguous locations with multiple result presentation
4. Provide confidence scoring for resolved locations
5. Cache frequently accessed location resolutions

### Intelligent Place Search
1. Apply location bias with configurable radius for local relevance
2. Support single-type filtering per Google Places API constraints
3. Implement rating, price level, and operational status filtering
4. Handle pagination with next page token management
5. Optimize search parameters for relevant results

### Local Proxy Architecture
1. Run FastAPI server on localhost for low-latency access
2. Proxy Google Places API requests with API key management
3. Transform API responses into consistent format
4. Implement request/response caching for performance
5. Handle rate limiting and error recovery

### Enhanced User Experience
1. Present search results with essential information display
2. Support iterative search refinement with feedback loops
3. Handle multiple location matches with user selection
4. Provide contextual suggestions for search improvement

## Data Structures

### Location Resolution Request
```typescript
interface LocationResolveRequest {
  location_text: string;       // natural language location description
  limit: number;               // maximum results (1-10)
  language?: string;           // preferred language for results
  region?: string;             // country/region bias
}
```

### Place Search Request
```typescript
interface PlaceSearchRequest {
  query: string;               // search query text
  location_bias: LocationBias; // geographic search center
  filters: SearchFilters;      // filtering criteria
  limit: number;               // result count limit (1-20)
  page_token?: string;         // pagination token
}

interface LocationBias {
  lat: number;                 // latitude coordinate
  lng: number;                 // longitude coordinate
  radius_m: number;            // search radius in meters (> 0)
}
```

### Search Filters
```typescript
interface SearchFilters {
  types?: string;              // single place type only
  price_levels?: number[];     // price range 0-4 (0=free, 4=very expensive)
  min_rating?: number;         // minimum rating 0-5 (0.5 increments)
  open_now?: boolean;          // currently open filter
  max_price_level?: number;    // maximum price threshold
}
```

### Place Result
```typescript
interface PlaceResult {
  place_id: string;            // Google Places unique identifier
  name: string;                // business/location name
  address: string;             // formatted address
  location: {
    lat: number;
    lng: number;
  };
  rating?: number;             // user rating (0-5)
  price_level?: number;        // price category (0-4)
  types: string[];             // place type categories
  open_now?: boolean;          // current operational status
  photos?: PlacePhoto[];       // associated images
  business_status?: string;    // operational status detail
}
```

### Search Response
```typescript
interface SearchResponse {
  results: PlaceResult[];      // matching places
  next_page_token?: string;    // pagination continuation
  total_count?: number;        // approximate total results
  status: 'OK' | 'ZERO_RESULTS' | 'INVALID_REQUEST' | 'OVER_QUERY_LIMIT';
}
```

### Server Configuration
```typescript
interface ServerConfig {
  host: string;                // server bind address (127.0.0.1)
  port: number;                // server port (8000)
  apiKey: string;              // Google Places API key
  cacheEnabled: boolean;       // response caching flag
  cacheTtl: number;            // cache time-to-live seconds
  rateLimitRpm: number;        // requests per minute limit
}
```

## Constraints

### Google Places API Limitations
- **Single Type Filter**: API accepts only one place type per search request
- **API Key Required**: Valid Google Places API key mandatory for all operations
- **Rate Limits**: Google-imposed request frequency and quota restrictions
- **Geographic Coverage**: Service availability varies by region

### Server and Performance Constraints
- **Local Hosting**: Runs on localhost requiring local server management
- **Memory Usage**: Caching and request processing consume local resources
- **Concurrent Limits**: Limited concurrent request handling capacity
- **Network Dependency**: Requires stable internet for Google API access

### Search and Filter Constraints
- **Radius Requirements**: Location bias radius must be greater than zero
- **Pagination Limits**: Google API pagination tokens expire after time
- **Price Level Mapping**: Discrete price levels 0-4 with specific meanings
- **Rating Granularity**: Minimum rating filters in 0.5 increments only

### Environment and Setup Requirements
- **Python Environment**: Requires UV package manager and virtual environment
- **Environment Variables**: GOOGLE_PLACES_API_KEY must be configured
- **Port Availability**: Port 8000 must be available for server binding
- **File System**: Write access for virtual environment and cache storage

## Edge Cases

### Location Resolution Problems
- **Ambiguous Locations**: Multiple valid interpretations of location text
- **Unknown Locations**: Location text not recognized by geocoding service
- **Coordinate Precision**: Low-confidence geocoding results
- **Language Mismatches**: Location names in unsupported languages
- **Historic Names**: Obsolete place names not current in mapping data

### Search and API Issues
- **Zero Results**: Search queries returning no matching places
- **API Quota Exceeded**: Google Places API usage limits reached
- **Service Outages**: Google Places API temporarily unavailable
- **Invalid Coordinates**: Malformed latitude/longitude values
- **Token Expiry**: Pagination tokens becoming invalid between requests

### Server and Network Problems
- **Server Startup Failures**: Port conflicts or missing dependencies
- **API Key Issues**: Invalid, expired, or restricted API credentials
- **Network Connectivity**: Internet connection loss affecting API access
- **Cache Corruption**: Local cache data becoming invalid or corrupted
- **Resource Exhaustion**: Server running out of memory or file descriptors

### Filter and Parameter Edge Cases
- **Invalid Price Levels**: Price level values outside 0-4 range
- **Extreme Radius Values**: Very large or very small search radius
- **Type Conflicts**: Attempting multiple place types in single request
- **Rating Out of Range**: Rating values outside 0-5 scale
- **Pagination Overflow**: Requesting more results than available

## Technical Debt

### API Integration Complexity
- Direct proxy approach without sophisticated error handling
- Limited retry logic for transient Google API failures
- No circuit breaker pattern for API reliability
- Basic rate limiting without intelligent throttling

### Caching and Performance
- Simple in-memory caching without persistence across restarts
- No cache invalidation strategy for stale location data
- Limited performance optimization for high-frequency requests
- No request deduplication for identical concurrent queries

### User Experience Limitations
- Command-line focused without web interface option
- Basic error messages without user-friendly explanations
- No search suggestion or autocomplete capabilities
- Limited integration with mapping or visualization tools

### Configuration and Deployment
- Manual server startup without service management
- Environment variable configuration without validation
- No monitoring or logging infrastructure
- Basic deployment model without containerization support

## Dependencies

### Core Dependencies
- **FastAPI**: Web framework for HTTP API server
- **UV Package Manager**: Python dependency management and virtual environments
- **Google Places API**: External service for location and place data
- **Uvicorn**: ASGI server for FastAPI application hosting

### Development Dependencies
- **Python 3.8+**: Programming language runtime requirement
- **Virtual Environment**: Isolated Python package environment
- **Environment File**: `.env` file for configuration management
- **HTTP Client**: For Google Places API communication

### System Dependencies
- **Network Connectivity**: Internet access for Google API communication
- **File System**: Configuration file and cache storage
- **Port Management**: Available port for HTTP server binding
- **Process Management**: Server process lifecycle management

### Google API Dependencies
- **Google Places API**: Location search and geocoding services
- **Google Cloud Console**: API key management and billing
- **Places API (New)**: Modern version of Google Places services
- **Geocoding API**: Location resolution and coordinate conversion

### Optional Dependencies
- **Redis**: Alternative caching backend for production use
- **Docker**: Containerization for deployment consistency
- **Nginx**: Reverse proxy for production deployment
- **Monitoring**: Application performance monitoring and logging