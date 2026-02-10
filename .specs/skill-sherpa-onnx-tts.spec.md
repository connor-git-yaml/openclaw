# Sherpa-ONNX-TTS Skill Specification

## Intent

Provide fully offline text-to-speech capabilities using the sherpa-onnx runtime with local voice models. Enables privacy-preserving speech synthesis without internet connectivity, cloud dependencies, or API costs using cross-platform ONNX neural network models.

## Interface

### Core Operations
- **Offline Speech Synthesis**: Convert text to speech using local ONNX models
- **Multi-Platform Support**: Cross-platform operation on macOS, Linux, and Windows
- **Voice Model Management**: Configurable voice models with different languages and styles
- **Audio File Generation**: WAV audio output with configurable quality settings

### Command Structure
- `sherpa-onnx-tts -o OUTPUT.wav "TEXT"` - Basic text-to-speech synthesis
- `sherpa-onnx-tts --model-file MODEL.onnx -o OUTPUT.wav "TEXT"` - Custom model
- `sherpa-onnx-tts --tokens-file TOKENS --data-dir DATA -o OUTPUT.wav "TEXT"` - Advanced config
- Runtime via wrapper script in `bin/sherpa-onnx-tts` with environment configuration

### Installation Components
- **Runtime Download**: Platform-specific sherpa-onnx binaries and libraries
- **Voice Models**: VITS-Piper models for different languages and speakers
- **Environment Configuration**: Runtime and model directory specification
- **Wrapper Script**: Simplified interface for command execution

### Model Support
- **VITS-Piper Models**: High-quality neural voice synthesis models
- **Multi-Language**: Various language and accent combinations
- **Quality Levels**: High/medium/low quality models for different use cases
- **Speaker Variety**: Multiple speaker options per language

## Business Logic

### Runtime Management and Configuration
1. Download and manage platform-specific sherpa-onnx runtime binaries
2. Configure environment variables for runtime and model locations
3. Handle cross-platform path resolution and binary execution
4. Validate runtime integrity and compatibility

### Voice Model Lifecycle
1. Download and extract ONNX voice models from repository releases
2. Organize models in structured directory hierarchy
3. Validate model compatibility with runtime version
4. Support multiple model configurations and switching

### Synthesis Pipeline
1. Process input text for optimal synthesis quality
2. Load specified ONNX model and associated data files
3. Execute neural network inference for speech generation
4. Output high-quality WAV audio files

### Cross-Platform Compatibility
1. Handle platform-specific binary execution (macOS/Linux/Windows)
2. Manage different archive formats and extraction methods
3. Resolve file paths and permissions across operating systems
4. Provide consistent interface regardless of platform

## Data Structures

### Runtime Configuration
```typescript
interface SherpaONNXConfig {
  runtimeDir: string;          // sherpa-onnx runtime directory
  modelDir: string;            // voice model directory
  modelFile?: string;          // specific ONNX model file
  tokensFile?: string;         // tokens file override
  dataDir?: string;            // model data directory
  platform: 'darwin' | 'linux' | 'win32';
  architecture: string;        // system architecture
}
```

### Voice Model
```typescript
interface VoiceModel {
  name: string;                // model identifier
  language: string;            // language code (e.g., en_US)
  speaker: string;             // speaker name (e.g., lessac)
  quality: 'high' | 'medium' | 'low';
  modelFile: string;           // ONNX model file path
  tokensFile: string;          // tokens file path
  dataDir: string;             // supporting data directory
  sampleRate: number;          // audio sample rate
  channels: number;            // audio channels
  downloadUrl: string;         // model download source
  size: number;                // model file size
}
```

### Synthesis Request
```typescript
interface SynthesisRequest {
  text: string;                // input text content
  outputFile: string;          // output WAV file path
  model?: string;              // specific model to use
  speed?: number;              // speech rate adjustment
  pitch?: number;              // pitch adjustment
  volume?: number;             // volume adjustment
  quality: 'high' | 'medium' | 'low';
  timeout?: number;            // synthesis timeout
}
```

### Audio Output
```typescript
interface AudioOutput {
  file: string;                // generated audio file path
  duration: number;            // audio duration in seconds
  sampleRate: number;          // audio sample rate
  channels: number;            // audio channel count
  bitDepth: number;            // audio bit depth
  fileSize: number;            // file size in bytes
  format: 'wav';               // output format
  quality: AudioQualityMetrics;
}

interface AudioQualityMetrics {
  snr?: number;                // signal-to-noise ratio
  clarity: number;             // perceived clarity score
  naturalness: number;         // naturalness rating
  intelligibility: number;     // speech intelligibility
}
```

### Platform Installation
```typescript
interface PlatformInstallation {
  platform: 'darwin' | 'linux' | 'win32';
  architecture: string;        // x64, universal2, etc.
  runtimeUrl: string;          // download URL for runtime
  runtimeVersion: string;      // sherpa-onnx version
  extractMethod: 'tar.bz2' | 'zip';
  targetDirectory: string;     // installation target
  executablePath: string;      // runtime binary path
  libraryPaths: string[];      // required library files
}
```

### Model Metadata
```typescript
interface ModelMetadata {
  modelFamily: 'vits-piper';   // model architecture type
  trainingDataset: string;     // training data source
  licenseType: string;         // usage license
  supportedLanguages: string[]; // language codes
  speakers: SpeakerInfo[];     // available speakers
  qualityLevels: QualityLevel[];
  technicalSpecs: TechnicalSpecs;
}

interface SpeakerInfo {
  id: string;                  // speaker identifier
  name: string;                // speaker display name
  gender: 'male' | 'female' | 'neutral';
  accent: string;              // accent description
  ageRange: string;            // apparent age range
  characteristics: string[];   // voice characteristics
}
```

## Constraints

### Platform and System Requirements
- **Operating System**: Limited to macOS, Linux, and Windows x64 platforms
- **Runtime Dependencies**: Requires sherpa-onnx runtime binaries for each platform
- **Storage Space**: Voice models range from hundreds of MB to several GB
- **Memory Requirements**: ONNX model inference requires sufficient RAM

### Model and Quality Limitations
- **Model Availability**: Limited to available VITS-Piper models in sherpa-onnx releases
- **Language Support**: Specific languages and accents based on trained models
- **Voice Variety**: Limited speaker options compared to cloud TTS services
- **Customization**: No real-time voice customization or personal voice training

### Audio Output Constraints
- **Format Support**: Limited to WAV audio output format
- **Quality Settings**: Fixed quality levels based on model training
- **Real-time Performance**: Synthesis speed depends on model complexity and hardware
- **File Size**: High-quality models generate large audio files

### Installation and Configuration Complexity
- **Manual Setup**: Requires manual runtime and model download and configuration
- **Environment Variables**: Complex environment variable configuration
- **Path Management**: Platform-specific path handling and resolution
- **Version Compatibility**: Runtime and model version compatibility requirements

## Edge Cases

### Installation and Setup Issues
- **Download Failures**: Network issues preventing runtime or model downloads
- **Extraction Problems**: Archive corruption or extraction permission issues
- **Path Configuration**: Incorrect environment variable configuration
- **Platform Mismatch**: Wrong runtime architecture for system
- **Disk Space**: Insufficient storage for runtime and model installation

### Runtime Execution Problems
- **Missing Dependencies**: Required libraries not available on system
- **Permission Denied**: Execute permissions not set on runtime binaries
- **Model Loading Failures**: Corrupted or incompatible voice models
- **Memory Exhaustion**: Large models consuming all available system memory
- **Process Timeout**: Long synthesis operations exceeding timeout limits

### Model and Voice Issues
- **Model Not Found**: Specified model file missing or inaccessible
- **Incompatible Model**: Model version not compatible with runtime
- **Poor Audio Quality**: Model producing distorted or unclear speech
- **Language Mismatch**: Text language not matching model language
- **Pronunciation Errors**: Model mispronouncing technical terms or names

### File System and Output Problems
- **Output Permission**: Unable to write audio files to specified location
- **File Size Limits**: Generated audio exceeding file system limits
- **Concurrent Access**: Multiple synthesis processes conflicting
- **Temporary Files**: Cleanup failures leaving temporary files
- **Path Length**: Very long file paths causing system issues

## Technical Debt

### Installation and Setup Complexity
- Manual download and configuration process requiring technical expertise
- No automated installation or update mechanisms
- Complex environment variable management across platforms
- Limited validation of installation integrity

### Model Management Limitations
- No centralized model repository or discovery mechanism
- Manual model download and organization required
- No model performance comparison or recommendation system
- Basic model metadata without detailed capability information

### Runtime Interface Gaps
- Basic command-line interface without advanced configuration options
- No streaming synthesis for real-time applications
- Limited error reporting and debugging information
- No progress indication for long synthesis operations

### Cross-Platform Compatibility Issues
- Platform-specific wrapper scripts and execution paths
- Inconsistent behavior across different operating systems
- Limited testing on diverse hardware configurations
- No graceful degradation for unsupported platforms

## Dependencies

### Core Dependencies
- **sherpa-onnx Runtime**: Platform-specific neural network inference engine
- **ONNX Models**: Voice synthesis models for speech generation
- **Archive Extraction**: tar.bz2 and zip extraction capabilities
- **File System Operations**: Download, extraction, and file management

### Platform-Specific Dependencies
- **macOS**: Universal2 binaries for Intel and Apple Silicon compatibility
- **Linux**: x64 shared libraries and runtime dependencies
- **Windows**: x64 binaries and Visual C++ runtime dependencies
- **Cross-Platform**: Consistent interface abstractions

### Neural Network Dependencies
- **ONNX Runtime**: Neural network model execution framework
- **Model Files**: .onnx voice model files with associated data
- **Tokenization**: Text tokenization and preprocessing components
- **Audio Processing**: WAV audio generation and encoding libraries

### Environment Dependencies
- **Environment Variables**: SHERPA_ONNX_RUNTIME_DIR and SHERPA_ONNX_MODEL_DIR
- **Path Resolution**: Cross-platform file path handling
- **Process Execution**: Command-line interface and process management
- **Configuration Storage**: OpenClaw configuration integration

### Optional Dependencies
- **Audio Playback**: System audio capabilities for direct playback
- **Text Processing**: Advanced text preprocessing for better synthesis
- **Model Conversion**: Tools for custom model training and conversion
- **Performance Monitoring**: Synthesis speed and quality measurement tools

### Development Dependencies
- **Build Tools**: Cross-platform wrapper script development
- **Testing Framework**: Synthesis quality and compatibility testing
- **Model Validation**: Voice model integrity and performance testing
- **Documentation**: Installation guides and troubleshooting resources