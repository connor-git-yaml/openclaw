# OpenAI Whisper API Skill Specification

## Intent

Provide cloud-based speech-to-text transcription through OpenAI's Whisper API service. Enables high-quality audio transcription with speaker identification, language detection, and structured output formats using OpenAI's hosted infrastructure.

## Interface

### Core Operations
- **Audio Transcription**: Convert speech to text using OpenAI's Whisper-1 model
- **Language Processing**: Automatic language detection and manual language specification
- **Context Enhancement**: Use prompts for speaker names and specialized terminology
- **Format Options**: Output in plain text or structured JSON format

### Script Interface
- `transcribe.sh AUDIO_FILE` - Basic transcription with default settings
- `transcribe.sh AUDIO_FILE --model whisper-1 --out OUTPUT.txt` - Custom output
- `transcribe.sh AUDIO_FILE --language en` - Language specification
- `transcribe.sh AUDIO_FILE --prompt "Speaker names: A, B"` - Context prompting
- `transcribe.sh AUDIO_FILE --json --out OUTPUT.json` - Structured output

### Configuration Methods
- **Environment Variable**: `OPENAI_API_KEY` for direct authentication
- **OpenClaw Config**: `skills."openai-whisper-api".apiKey` in configuration file
- **Default Model**: `whisper-1` (OpenAI's production Whisper model)
- **Output Defaults**: `<input_filename>.txt` for transcription results

### API Integration
- **Endpoint**: `/v1/audio/transcriptions` for transcription operations
- **HTTP Method**: POST with multipart form data for audio upload
- **Authentication**: Bearer token authentication with OpenAI API key
- **File Upload**: Direct audio file upload to OpenAI infrastructure

## Business Logic

### API Request Management
1. Validate audio file format and size constraints
2. Prepare multipart form data with audio file and parameters
3. Execute authenticated HTTPS request to OpenAI API endpoint
4. Handle API rate limiting and quota management
5. Process API response and extract transcription results

### Audio File Processing
1. Accept various audio formats supported by OpenAI (mp3, mp4, mpeg, mpga, m4a, wav, webm)
2. Validate file size limitations (25MB maximum)
3. Handle file path resolution and access permissions
4. Stream large audio files efficiently for upload

### Configuration and Authentication
1. Resolve API key from multiple configuration sources
2. Validate API key format and accessibility
3. Handle authentication errors and key rotation
4. Support both environment variables and OpenClaw configuration

### Response Processing and Output
1. Parse JSON responses from OpenAI API
2. Extract transcription text and metadata
3. Format output according to specified format (text/JSON)
4. Write results to specified output file or default location

### Error Handling and Recovery
1. Handle network connectivity issues and timeouts
2. Process API errors (authentication, quota, format issues)
3. Provide meaningful error messages for troubleshooting
4. Implement retry logic for transient failures

## Data Structures

### Transcription Request
```typescript
interface TranscriptionRequest {
  audioFile: string;           // path to audio file
  model: 'whisper-1';          // OpenAI model identifier
  language?: string;           // ISO 639-1 language code
  prompt?: string;             // context prompt for better accuracy
  responseFormat: 'json' | 'text'; // output format preference
  temperature?: number;        // generation randomness (0-1)
  outputPath?: string;         // custom output file location
}
```

### API Configuration
```typescript
interface WhisperAPIConfig {
  apiKey: string;              // OpenAI API authentication key
  baseUrl: string;             // API endpoint base URL
  model: string;               // default model name
  maxFileSize: number;         // maximum file size in bytes (25MB)
  timeout: number;             // request timeout in seconds
  retryAttempts: number;       // retry count for failed requests
  rateLimitDelay: number;      // delay between requests for rate limiting
}
```

### Transcription Response
```typescript
interface TranscriptionResponse {
  text: string;                // complete transcription text
  language?: string;           // detected language (if not specified)
  duration?: number;           // audio duration in seconds
  segments?: TranscriptionSegment[]; // word/phrase level segments (JSON format)
  confidence?: number;         // overall transcription confidence
  processingTime: number;      // API processing duration
}

interface TranscriptionSegment {
  id: number;                  // segment identifier
  seek: number;                // segment position in audio
  start: number;               // start time in seconds
  end: number;                 // end time in seconds
  text: string;                // segment text content
  tokens: number[];            // token array for segment
  temperature: number;         // generation temperature used
  avgLogprob: number;          // average log probability
  compressionRatio: number;    // compression ratio indicator
  noSpeechProb: number;        // probability of no speech
}
```

### Script Operation
```typescript
interface ScriptOperation {
  inputFile: string;           // source audio file path
  outputFile: string;          // destination transcription file
  parameters: ScriptParameters; // command-line options
  apiRequest: TranscriptionRequest; // formatted API request
  result?: TranscriptionResponse; // operation result
  errors?: string[];           // error messages
  executionTime: number;       // total script execution time
}

interface ScriptParameters {
  model?: string;              // model override
  language?: string;           // language specification
  prompt?: string;             // context prompt
  outputFormat: 'text' | 'json'; // output format
  outputPath?: string;         // custom output location
}
```

### Error Response
```typescript
interface APIError {
  error: {
    message: string;           // error description
    type: string;              // error category
    param?: string;            // parameter causing error
    code?: string;             // error code identifier
  };
  statusCode: number;          // HTTP status code
  requestId?: string;          // request identifier for debugging
}
```

## Constraints

### OpenAI API Limitations
- **File Size**: Maximum 25MB per audio file
- **Supported Formats**: mp3, mp4, mpeg, mpga, m4a, wav, webm only
- **Rate Limits**: API requests subject to account-based rate limiting
- **Model Availability**: Limited to OpenAI's available Whisper models

### Authentication Requirements
- **API Key**: Valid OpenAI API key with sufficient credits/quota
- **Account Status**: Active OpenAI account in good standing
- **Billing**: Sufficient account balance or payment method for usage charges
- **Usage Limits**: API usage subject to account tier limitations

### Network Dependencies
- **Internet Connectivity**: Stable internet connection required for API access
- **Upload Bandwidth**: Sufficient bandwidth for audio file upload
- **API Availability**: Dependent on OpenAI service uptime and performance
- **Latency**: Network latency affects total processing time

### Audio Quality Requirements
- **Format Support**: Audio must be in supported format
- **Quality Standards**: Better audio quality improves transcription accuracy
- **Length Limitations**: Very long audio files may hit processing timeouts
- **Encoding**: Properly encoded audio files for optimal results

## Edge Cases

### API and Authentication Issues
- **Invalid API Key**: Expired, revoked, or malformed authentication credentials
- **Quota Exceeded**: Account usage limits reached preventing further requests
- **Rate Limiting**: Too many concurrent or rapid requests causing throttling
- **Service Outage**: OpenAI API temporarily unavailable or experiencing issues
- **Account Suspension**: OpenAI account suspended affecting API access

### File and Format Problems
- **Unsupported Format**: Audio files in formats not supported by API
- **File Too Large**: Audio files exceeding 25MB size limit
- **Corrupted Files**: Damaged or incomplete audio files
- **Permission Denied**: Unable to read input or write output files
- **Network Upload Failures**: File upload interruption or corruption

### Audio Content Challenges
- **Poor Audio Quality**: Background noise, low volume, or distorted audio
- **Multiple Languages**: Mixed language content in single audio file
- **Specialized Terminology**: Technical terms not recognized accurately
- **Multiple Speakers**: Overlapping speech or rapid speaker changes
- **Non-Speech Audio**: Music or sound effects confusing transcription

### Configuration and Environment Issues
- **Missing Configuration**: API key not set in environment or config file
- **Path Resolution**: Script unable to locate audio files or output directories
- **Permission Problems**: Insufficient permissions for file operations
- **Environment Conflicts**: Multiple configuration sources causing conflicts
- **Script Execution**: Shell script execution failures or dependency issues

## Technical Debt

### Error Handling Limitations
- Basic error reporting without specific recovery guidance
- Limited retry logic for different types of API failures
- No circuit breaker pattern for API reliability
- Insufficient error context for debugging complex issues

### Configuration Management
- Simple configuration model without validation or migration
- No secure credential storage beyond environment variables
- Limited configuration discovery and setup assistance
- Basic credential rotation and management capabilities

### Performance and Optimization
- No request batching for multiple audio files
- Basic upload progress tracking and user feedback
- Limited optimization for large file uploads
- No intelligent model selection based on audio characteristics

### Integration and Workflow
- Shell script approach limits cross-platform compatibility
- No integration with popular audio processing workflows
- Basic output formatting without advanced customization
- Limited metadata extraction and processing

## Dependencies

### Core Dependencies
- **curl**: HTTP client for API communication
- **Bash/Shell**: Script execution environment
- **OpenAI API**: Cloud-based Whisper transcription service

### Authentication Dependencies
- **OpenAI API Key**: Valid authentication credential for service access
- **OpenClaw Configuration**: Optional configuration system integration
- **Environment Variables**: System environment for credential storage

### System Dependencies
- **File System Access**: Read access to audio files, write access to output directory
- **Network Stack**: HTTPS connectivity for API communication
- **Shell Environment**: Unix-like shell environment for script execution

### Audio Processing Dependencies
- **File Format Support**: System support for reading various audio formats
- **Path Resolution**: File system path handling and validation
- **Stream Processing**: Efficient file upload for large audio files

### Optional Dependencies
- **OpenClaw Framework**: Integration with OpenClaw skill configuration system
- **JSON Processing**: jq or similar tools for JSON output manipulation
- **Progress Tracking**: Enhanced user feedback for long operations
- **Batch Processing**: Tools for processing multiple audio files

### Development Dependencies
- **Testing Framework**: Validation of transcription accuracy and API integration
- **Documentation**: API reference and usage example maintenance
- **Error Simulation**: Testing of various failure scenarios
- **Performance Monitoring**: API usage tracking and optimization analysis