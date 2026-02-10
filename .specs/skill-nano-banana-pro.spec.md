# Nano Banana Pro Skill Specification

## Intent

Provide advanced image generation and editing capabilities through Gemini 3 Pro Image integration. Enables AI-powered image creation, single-image editing, multi-image composition, and resolution-controlled output for creative and practical image manipulation workflows.

## Interface

### Core Operations
- **Image Generation**: Create new images from text prompts
- **Single Image Editing**: Modify existing images based on text instructions
- **Multi-Image Composition**: Combine multiple images (up to 14) into unified scenes
- **Resolution Control**: Generate images at 1K, 2K, or 4K resolutions

### Command Structure
- `uv run generate_image.py --prompt "DESCRIPTION" --filename OUTPUT.png` - Generate image
- `uv run generate_image.py --prompt "EDIT_INSTRUCTIONS" -i INPUT.png` - Edit image
- `uv run generate_image.py --prompt "COMBINE_INSTRUCTIONS" -i IMG1 -i IMG2` - Multi-image
- `uv run generate_image.py --resolution 2K --filename timestamped.png` - High resolution

### Input Methods
- **Text Prompts**: Natural language descriptions for generation or editing
- **Single Image Input**: Image file path for editing operations
- **Multiple Image Input**: Up to 14 images for composition workflows
- **Resolution Specification**: 1K (default), 2K, 4K output resolution options

### Output Integration
- **File Output**: PNG format with configurable naming and timestamping
- **Media Integration**: MEDIA: line output for OpenClaw chat attachment
- **Path Reporting**: Generated file path for subsequent workflow integration

## Business Logic

### Image Generation Pipeline
1. Accept natural language image description prompts
2. Process prompts through Gemini 3 Pro Image API
3. Generate images at specified resolution and quality
4. Save output with organized filename conventions
5. Report generation results for workflow integration

### Image Editing Workflow
1. Load existing image for modification context
2. Apply text-based editing instructions via AI model
3. Preserve original image aspects while applying changes
4. Generate edited result maintaining quality and resolution
5. Output modified image with clear change documentation

### Multi-Image Composition System
1. Accept up to 14 input images for composition operations
2. Analyze input images for composition compatibility
3. Apply composition instructions to merge images coherently
4. Handle different image sizes and formats during composition
5. Generate unified output maintaining visual quality

### Resolution and Quality Management
1. Support multiple output resolutions (1K, 2K, 4K)
2. Optimize generation parameters for target resolution
3. Balance quality with generation time and resource usage
4. Handle resolution constraints based on input image dimensions

## Data Structures

### Generation Request
```typescript
interface GenerationRequest {
  prompt: string;              // image description or editing instruction
  filename: string;            // output file path
  resolution: '1K' | '2K' | '4K'; // target resolution
  mode: 'generate' | 'edit' | 'compose';
  inputImages?: string[];      // paths to input images (editing/composition)
  outputFormat: 'png';         // output file format
  quality?: number;            // output quality setting
}
```

### Image Processing Context
```typescript
interface ProcessingContext {
  requestId: string;           // unique operation identifier
  apiKey: string;              // Gemini API authentication
  model: 'gemini-3-pro-image'; // AI model specification
  timestamp: Date;             // operation timestamp
  workingDirectory: string;    // temporary file location
  inputValidation: ValidationResult[];
}
```

### Generation Result
```typescript
interface GenerationResult {
  success: boolean;            // operation success status
  outputPath: string;          // generated image file path
  resolution: string;          // actual output resolution
  fileSize: number;            // output file size in bytes
  processingTime: number;      // generation time in seconds
  mediaLine: string;           // MEDIA: output for OpenClaw integration
  error?: string;              // error message if failed
}
```

### Input Image Metadata
```typescript
interface InputImageInfo {
  path: string;                // image file path
  dimensions: {
    width: number;
    height: number;
  };
  format: string;              // image format (PNG, JPEG, etc.)
  size: number;                // file size in bytes
  isValid: boolean;            // format and accessibility validation
}
```

### API Configuration
```typescript
interface GeminiConfig {
  apiKey: string;              // Gemini API authentication key
  endpoint: string;            // API endpoint URL
  model: string;               // model identifier
  timeout: number;             // request timeout seconds
  retryAttempts: number;       // retry count for failed requests
  maxInputImages: number;      // maximum images for composition (14)
}
```

## Constraints

### Google Gemini API Limitations
- **API Key Required**: Valid Gemini API key mandatory for all operations
- **Usage Quotas**: Daily and per-minute request limits
- **Image Limits**: Maximum 14 images for multi-image composition
- **File Size Limits**: Input image size restrictions

### Resolution and Quality Constraints
- **Output Resolutions**: Limited to 1K, 2K, 4K predefined options
- **Processing Time**: Higher resolutions require significantly more processing time
- **Memory Usage**: Large resolution operations consume substantial system memory
- **Disk Space**: High-resolution outputs require significant storage space

### Input Format Requirements
- **Supported Formats**: PNG, JPEG input formats for editing operations
- **File Accessibility**: Input images must be readable from specified paths
- **Image Validity**: Input images must be valid and non-corrupted
- **Path Limitations**: File path length and character restrictions

### System Dependencies
- **UV Package Manager**: Required for Python environment and dependency management
- **Python Runtime**: Compatible Python version for script execution
- **File System Access**: Read/write permissions for input and output directories
- **Network Connectivity**: Internet access for Gemini API communication

## Edge Cases

### API and Service Issues
- **API Key Invalid**: Expired, restricted, or malformed authentication credentials
- **Service Outage**: Gemini API temporarily unavailable or rate limited
- **Quota Exceeded**: Daily usage limits reached preventing operations
- **Request Timeout**: Long-running operations exceeding timeout limits
- **Model Unavailable**: Gemini 3 Pro Image model temporarily offline

### Input and File Problems
- **Missing Input Files**: Specified input images not found or inaccessible
- **Corrupted Images**: Input images with format corruption or damage
- **Unsupported Formats**: Image formats not supported by processing pipeline
- **Large File Issues**: Oversized input images causing processing failures
- **Permission Denied**: Insufficient file system permissions for input/output

### Processing and Generation Failures
- **Memory Exhaustion**: High-resolution operations consuming all available memory
- **Disk Space Full**: Output generation failing due to insufficient storage
- **Processing Timeout**: Complex operations exceeding reasonable time limits
- **Quality Degradation**: Generated images not meeting expected quality standards
- **Composition Conflicts**: Multi-image inputs with incompatible characteristics

### Configuration and Environment Issues
- **Missing Dependencies**: UV or Python environment not properly configured
- **Environment Variable Issues**: API key not set or accessible
- **Working Directory Problems**: Temporary file location inaccessible or full
- **Path Resolution**: File path handling issues across different operating systems
- **Script Execution**: Python script failing to run due to environment issues

## Technical Debt

### Error Handling and Recovery
- Basic error reporting without specific guidance for different failure types
- Limited retry logic for transient API failures
- No graceful degradation when high resolutions unavailable
- Poor error context for file system and permission issues

### Performance and Resource Management
- No optimization for batch processing multiple requests
- Memory usage not optimized for large image operations
- No caching of intermediate results for similar operations
- Limited concurrent processing capabilities

### User Experience and Integration
- Command-line only interface without graphical previews
- Basic filename conventions without advanced organization
- Limited integration with image editing and management tools
- No interactive parameter tuning for quality optimization

### Feature Completeness
- No support for additional output formats beyond PNG
- Limited control over generation parameters and style
- No support for progressive enhancement or iterative editing
- Missing integration with popular image processing workflows

## Dependencies

### Core Dependencies
- **UV Package Manager**: Python environment and dependency management
- **Python Runtime**: Script execution environment
- **Gemini API Client**: Google AI API integration library
- **Image Processing**: PIL/Pillow for image format handling

### Google API Dependencies
- **Gemini 3 Pro Image**: AI model for image generation and editing
- **Google AI Platform**: Authentication and billing infrastructure
- **API Key Management**: Valid Gemini API credentials

### System Dependencies
- **File System**: Input/output file operations and temporary storage
- **Network Stack**: HTTPS connectivity for API communication
- **Memory Management**: RAM for image processing and generation
- **Python Environment**: Virtual environment for dependency isolation

### Image Processing Dependencies
- **PIL/Pillow**: Python imaging library for format conversion
- **Image Validation**: File format verification and metadata extraction
- **Resolution Processing**: Image scaling and dimension handling
- **Format Conversion**: PNG output format generation

### Optional Dependencies
- **OpenClaw Integration**: MEDIA: line output for chat attachment
- **Timestamp Utilities**: Filename generation with datetime stamps
- **Configuration Management**: Skill-specific configuration in OpenClaw
- **Logging Framework**: Operation logging and debugging support

### Development Dependencies
- **UV Build System**: Package management and virtual environment
- **Type Checking**: Static analysis for Python code quality
- **Testing Framework**: Validation of image generation functionality
- **API Documentation**: Gemini API reference and usage examples