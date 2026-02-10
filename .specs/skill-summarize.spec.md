# Summarize Skill Specification

## Intent

Extract and summarize content from URLs, podcasts, videos, and local files. Serves as comprehensive fallback for transcribing YouTube videos and other media, providing text extraction and intelligent summarization capabilities.

## Interface

### Core Operations
- **URL Summarization**: Extract and summarize web page content
- **Video Processing**: Transcribe and summarize YouTube and other video content
- **Podcast Processing**: Audio transcription and content extraction
- **File Processing**: Local file content analysis and summarization
- **Multi-format Support**: Handle various media and document formats

### Content Types
- **Web Pages**: HTML content extraction and analysis
- **Video Content**: YouTube, Vimeo, and other video platforms
- **Audio Files**: Podcasts, recordings, and audio content
- **Documents**: PDFs, text files, and structured documents

## Business Logic

### Content Extraction and Processing
1. Identify content type and select appropriate extraction method
2. Download or access content while respecting rate limits
3. Extract text from various media formats using specialized tools
4. Clean and preprocess extracted content for analysis

### Intelligent Summarization
1. Apply natural language processing for content analysis
2. Generate concise summaries maintaining key information
3. Handle different content styles and domains appropriately
4. Provide structured output with key points and themes

### Multi-modal Content Handling
1. Process video content by extracting audio for transcription
2. Handle audio files directly with speech-to-text systems
3. Extract text from PDFs and documents with OCR when needed
4. Combine multiple content sources for comprehensive analysis

## Data Structures

### Content Source
```typescript
interface ContentSource {
  type: 'url' | 'video' | 'audio' | 'file';
  url?: string; // web URL or video URL
  path?: string; // local file path
  format: string; // content format/MIME type
  metadata: SourceMetadata; // content metadata
}

interface SourceMetadata {
  title?: string; // content title
  author?: string; // content author/creator
  duration?: number; // media duration (seconds)
  published?: Date; // publication date
  size?: number; // file/content size
}
```

### Extraction Result
```typescript
interface ExtractionResult {
  source: ContentSource; // original content source
  text: string; // extracted text content
  confidence: number; // extraction confidence score
  language?: string; // detected language
  timestamp: Date; // extraction time
  method: string; // extraction method used
}
```

### Summary Output
```typescript
interface Summary {
  original: ExtractionResult; // source content
  summary: string; // generated summary
  keyPoints: string[]; // main points extracted
  themes: string[]; // identified themes
  length: 'brief' | 'detailed'; // summary length
  confidence: number; // summarization quality score
}
```

## Constraints

### Content Access Limitations
- **Platform Restrictions**: Some platforms block automated content access
- **Rate Limiting**: API and scraping rate limits
- **Geographical Blocks**: Content not available in certain regions
- **Authentication**: Login-required content not accessible

### Processing Performance
- **Large Files**: Memory and processing time for large content
- **Video Processing**: CPU-intensive transcription operations
- **Network Bandwidth**: Download speed affects processing time
- **API Costs**: Third-party services may incur usage costs

### Quality and Accuracy
- **Transcription Quality**: Audio quality affects transcription accuracy
- **Language Support**: Limited language support for some content types
- **Domain Knowledge**: Technical content may be poorly summarized
- **Context Loss**: Important nuances may be lost in summarization

## Edge Cases

### Content Access Issues
- **Blocked URLs**: Sites blocking automated access
- **Expired Content**: Videos or pages no longer available
- **Private Content**: Content requiring authentication
- **Broken Links**: Invalid or redirected URLs

### Processing Failures
- **Format Unsupported**: Unknown or proprietary content formats
- **Corruption**: Damaged or incomplete content files
- **Size Limits**: Content exceeding processing capabilities
- **Encoding Issues**: Character encoding problems

### Quality Problems
- **Poor Audio**: Low-quality audio affecting transcription
- **Multiple Languages**: Mixed language content processing
- **Technical Content**: Specialized terminology handling
- **No Meaningful Content**: Empty or meaningless input

## Technical Debt

### Tool Integration Complexity
- Multiple specialized tools for different content types
- Inconsistent interfaces and error handling across tools
- Complex dependency management for media processing

### Content Type Detection
- Basic content type identification without advanced analysis
- Limited format support for emerging media types
- No automatic quality assessment for processing decisions

### Summarization Quality
- Generic summarization without domain-specific optimization
- Limited customization options for summary style and length
- No quality metrics or confidence scoring

## Dependencies

### Core Dependencies
- **summarize.sh**: Main summarization tool
- **Media Processing**: FFmpeg for audio/video extraction
- **Web Scraping**: HTTP clients and HTML parsing tools
- **Transcription**: Speech-to-text engines (Whisper, etc.)

### System Dependencies
- **Network Connectivity**: Internet access for URL content
- **Storage Space**: Temporary files for media processing
- **Processing Power**: CPU for transcription and analysis
- **Memory**: RAM for large content processing

### Optional Dependencies
- **API Services**: Cloud transcription and summarization APIs
- **OCR Tools**: Optical character recognition for image text
- **Language Detection**: Automatic language identification
- **Content Filtering**: Spam and quality filtering tools