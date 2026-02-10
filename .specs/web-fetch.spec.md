# Web Fetch Module Specification

## Intent
The Web Fetch module provides lightweight web content retrieval and extraction capabilities for OpenClaw agents. It enables fetching and processing of web pages without full browser automation, converting HTML to readable markdown or text format. The module supports content extraction, text processing, and efficient handling of various web content types while maintaining security and performance.

## Interface

### Core Types
```typescript
export type WebFetchRequest = {
  url: string;
  extractMode?: 'markdown' | 'text';
  maxChars?: number;
  headers?: Record<string, string>;
  timeout?: number;
  followRedirects?: boolean;
  userAgent?: string;
  encoding?: string;
};

export type WebFetchResult = {
  success: boolean;
  url: string;
  finalUrl: string;
  statusCode: number;
  headers: Record<string, string>;
  content: string;
  contentType: string;
  contentLength: number;
  extractedContent?: string;
  title?: string;
  description?: string;
  language?: string;
  publishDate?: Date;
  author?: string;
  error?: string;
};

export type ContentExtraction = {
  mode: 'markdown' | 'text';
  removeAds: boolean;
  preserveFormatting: boolean;
  extractMetadata: boolean;
  maxLength?: number;
  minLength?: number;
};
```

### Key Functions
- `fetchUrl(url, options)` - Fetch and extract content from URL
- `extractContent(html, mode)` - Extract readable content from HTML
- `validateUrl(url)` - Validate URL format and accessibility
- `getMetadata(html)` - Extract metadata from HTML content
- `convertToMarkdown(html)` - Convert HTML to markdown format
- `sanitizeContent(content)` - Clean and sanitize extracted content

### Extraction Modes
- **Markdown Mode**: Convert HTML to structured markdown
- **Text Mode**: Extract plain text content only
- **Metadata Extraction**: Extract title, description, author, date
- **Link Preservation**: Maintain important links in output
- **Image Handling**: Include or exclude image references

## Business Logic

### Content Fetching Process
1. **URL Validation**: Validate and normalize input URL
2. **Request Preparation**: Set appropriate headers and user agent
3. **HTTP Request**: Perform HTTP GET request with timeout handling
4. **Response Validation**: Check status code and content type
5. **Content Processing**: Extract and convert content based on mode
6. **Metadata Extraction**: Extract page metadata if requested
7. **Content Sanitization**: Remove unwanted elements and scripts
8. **Size Limiting**: Truncate content if it exceeds limits

### HTML Content Extraction
- **Main Content Detection**: Identify main article/content area
- **Noise Removal**: Remove ads, navigation, sidebars, comments
- **Text Extraction**: Extract readable text from HTML elements
- **Structure Preservation**: Maintain heading hierarchy and lists
- **Link Processing**: Handle internal and external links appropriately
- **Image Handling**: Process images based on configuration

### Markdown Conversion
- **Heading Conversion**: Convert HTML headings to markdown
- **List Processing**: Convert HTML lists to markdown format
- **Link Formatting**: Convert links to markdown link syntax
- **Table Handling**: Convert HTML tables to markdown tables
- **Code Block Processing**: Preserve code blocks and formatting
- **Emphasis Handling**: Convert bold, italic, and other emphasis

### Content Quality Assessment
- **Readability Scoring**: Assess content readability and quality
- **Language Detection**: Identify content language
- **Content Classification**: Classify content type (article, blog, news, etc.)
- **Spam Detection**: Identify and filter low-quality content
- **Duplicate Detection**: Identify duplicate or near-duplicate content

### Security and Safety
- **URL Filtering**: Block malicious or inappropriate URLs
- **Content Sanitization**: Remove potentially harmful content
- **Rate Limiting**: Prevent abuse and respect robots.txt
- **Privacy Protection**: Handle sensitive content appropriately
- **Error Handling**: Graceful handling of fetch failures

## Data Structures

### Fetch Cache Entry
```typescript
type FetchCacheEntry = {
  url: string;
  timestamp: Date;
  ttl: number;
  content: string;
  metadata: ContentMetadata;
  etag?: string;
  lastModified?: string;
};
```

### Content Metadata
```typescript
type ContentMetadata = {
  title?: string;
  description?: string;
  author?: string;
  publishDate?: Date;
  language?: string;
  keywords?: string[];
  canonicalUrl?: string;
  siteName?: string;
  contentType: string;
  wordCount: number;
  readingTime: number;
};
```

### Extraction Rules
```typescript
type ExtractionRules = {
  contentSelectors: string[];
  removeSelectors: string[];
  preserveSelectors: string[];
  titleSelectors: string[];
  authorSelectors: string[];
  dateSelectors: string[];
  languageHints: string[];
};
```

### Request Context
```typescript
type RequestContext = {
  url: string;
  originalUrl: string;
  redirectChain: string[];
  requestTime: Date;
  responseTime: Date;
  cacheHit: boolean;
  userAgent: string;
  referer?: string;
};
```

## Constraints

### Performance Constraints
- **Request Timeout**: Default 30 seconds, maximum 60 seconds
- **Content Size**: Maximum 10MB per request
- **Extraction Time**: Maximum 5 seconds for content processing
- **Memory Usage**: Content processing limited to 50MB memory

### Security Constraints
- **URL Validation**: Only HTTP/HTTPS URLs accepted
- **Content Filtering**: Block executable content and scripts
- **Rate Limiting**: Maximum 10 requests per minute per domain
- **User Agent**: Required user agent for all requests

### Content Processing Constraints
- **Character Limits**: Configurable maximum content length (default 50k chars)
- **Language Support**: UTF-8 encoding required for proper processing
- **Format Support**: Limited to HTML, XML, and plain text content
- **Extraction Quality**: Best effort extraction, not guaranteed perfect

### Network Constraints
- **Redirect Limits**: Maximum 5 redirects per request
- **DNS Resolution**: Standard DNS resolution timeout (5 seconds)
- **Connection Limits**: Maximum 3 concurrent connections per domain
- **Protocol Support**: HTTP/1.1 and HTTP/2 support only

## Edge Cases

### Network and Connectivity Issues
- **Connection Timeouts**: Handle slow or unresponsive servers
- **DNS Resolution Failures**: Graceful handling of invalid domains
- **SSL Certificate Issues**: Handle expired or invalid certificates
- **Network Interruptions**: Handle connection drops during fetch

### Content Processing Edge Cases
- **Malformed HTML**: Handle broken or invalid HTML gracefully
- **Encoding Issues**: Handle various character encodings properly
- **Large Content**: Handle very large pages without memory issues
- **Empty Content**: Handle pages with minimal or no content

### HTTP Response Edge Cases
- **Unexpected Status Codes**: Handle 4xx, 5xx, and other error codes
- **Redirect Loops**: Detect and break infinite redirect loops
- **Content-Type Mismatches**: Handle incorrect content-type headers
- **Partial Content**: Handle incomplete or truncated responses

### Security Edge Cases
- **Malicious Content**: Detect and handle potentially harmful content
- **Phishing Sites**: Identify and block known phishing domains
- **Large Binary Files**: Prevent downloading of large non-text files
- **Rate Limit Bypass**: Prevent attempts to bypass rate limiting

## Technical Debt

### Known Issues
- **Extraction Accuracy**: Content extraction not perfect for all sites
- **Memory Usage**: Large pages can cause memory spikes
- **Cache Implementation**: Simple in-memory cache not persistent
- **Error Messages**: Generic error messages lack specific details

### Performance Optimizations Needed
- **Streaming Processing**: Process content as it's received
- **Compression Support**: Add gzip/deflate response compression
- **Connection Pooling**: Reuse connections for better performance
- **Selective Parsing**: Only parse necessary parts of large documents

### Feature Gaps
- **JavaScript Rendering**: No support for JavaScript-heavy sites
- **Authentication**: Limited support for authenticated content
- **Cookie Handling**: No cookie persistence across requests
- **Proxy Support**: No proxy configuration options

### Architecture Improvements Required
- **Plugin System**: Extensible extraction rules and processors
- **Content Caching**: Persistent caching with expiration
- **Quality Metrics**: Better content quality assessment
- **Batch Processing**: Support for bulk URL processing

## Dependencies

### Internal Dependencies
- **Config**: Web fetch configuration and rate limiting settings
- **Security**: URL validation and content filtering
- **Logging**: Request and error logging
- **Tools**: Integration with agent tool system

### External Dependencies
- **HTTP Client**: Node.js fetch or axios for HTTP requests
- **HTML Parser**: cheerio or similar for DOM manipulation
- **Text Processing**: Natural language processing libraries
- **Content Extraction**: Readability or similar content extraction

### Optional Dependencies
- **Language Detection**: franc or similar language detection library
- **Metadata Extraction**: Open Graph and Twitter Card parsers
- **Content Classification**: ML libraries for content type detection
- **Image Processing**: Sharp or similar for image metadata extraction