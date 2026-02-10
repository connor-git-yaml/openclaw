# OpenAI Whisper Skill Specification

## Intent

Provide local speech-to-text transcription capabilities through OpenAI's Whisper CLI tool. Enables offline audio transcription, translation, and subtitle generation without API dependencies using locally downloaded AI models.

## Interface

### Core Operations
- **Audio Transcription**: Convert speech to text in original language
- **Audio Translation**: Translate foreign speech to English text
- **Format Generation**: Output in multiple formats (txt, srt, vtt, json)
- **Model Selection**: Choose from various model sizes for speed/accuracy trade-offs

### Command Structure
- `whisper AUDIO_FILE --model MODEL --output_format FORMAT` - Basic transcription
- `whisper AUDIO_FILE --task translate --output_format srt` - Translation with subtitles
- `whisper AUDIO_FILE --output_dir PATH --language LANG` - Directed output
- `whisper AUDIO_FILE --initial_prompt "CONTEXT"` - Context-aware transcription

### Model Options
- **turbo**: Default fast model for real-time processing
- **tiny**: Fastest processing with basic accuracy
- **base**: Balanced speed and accuracy for general use
- **small**: Better accuracy with moderate processing time
- **medium**: High accuracy for professional transcription
- **large**: Maximum accuracy for critical transcription tasks

### Output Formats
- **txt**: Plain text transcription
- **srt**: SubRip subtitle format with timestamps
- **vtt**: WebVTT subtitle format for web players
- **json**: Structured output with detailed metadata
- **tsv**: Tab-separated values with timestamps

## Business Logic

### Audio Processing Pipeline
1. Accept various audio formats (mp3, m4a, wav, flac, etc.)
2. Preprocess audio for optimal transcription quality
3. Segment audio into manageable chunks for processing
4. Apply selected Whisper model for speech recognition
5. Post-process results for accuracy and formatting

### Model Management System
1. Download models to local cache (`~/.cache/whisper`) on first use
2. Select appropriate model based on accuracy/speed requirements
3. Handle model loading and GPU/CPU resource allocation
4. Cache models for subsequent faster processing

### Language Detection and Processing
1. Automatically detect spoken language when not specified
2. Handle multilingual content with language switching
3. Apply language-specific transcription optimizations
4. Support translation from any language to English

### Context-Aware Transcription
1. Use initial prompts to guide transcription accuracy
2. Apply domain-specific terminology and context
3. Handle specialized vocabulary and technical terms
4. Maintain consistency across long audio sessions

## Data Structures

### Transcription Request
```typescript
interface TranscriptionRequest {
  audioFile: string;           // path to audio file
  model: WhisperModel;         // selected model size
  task: 'transcribe' | 'translate'; // operation type
  language?: string;           // source language code
  outputFormat: OutputFormat;  // desired output format
  outputDir?: string;          // output directory
  initialPrompt?: string;      // transcription context
  temperature?: number;        // generation randomness (0-1)
}
```

### Whisper Model
```typescript
interface WhisperModel {
  name: 'turbo' | 'tiny' | 'base' | 'small' | 'medium' | 'large';
  size: number;                // model size in MB
  isDownloaded: boolean;       // local availability
  downloadUrl: string;         // model download location
  accuracy: number;            // relative accuracy score
  processingSpeed: number;     // relative processing speed
  memoryRequirement: number;   // RAM requirement in MB
}
```

### Transcription Result
```typescript
interface TranscriptionResult {
  text: string;                // complete transcription
  language: string;            // detected/specified language
  segments: TranscriptionSegment[]; // timestamped segments
  processingTime: number;      // total processing duration
  confidence: number;          // overall transcription confidence
  outputFiles: string[];       // generated output file paths
}

interface TranscriptionSegment {
  id: number;                  // segment identifier
  start: number;               // start timestamp (seconds)
  end: number;                 // end timestamp (seconds)
  text: string;                // segment text content
  confidence: number;          // segment confidence score
  noSpeechProb: number;        // probability of no speech
}
```

### Output Configuration
```typescript
interface OutputConfig {
  format: OutputFormat;
  directory: string;           // output directory path
  filename?: string;           // custom filename prefix
  includeMetadata: boolean;    // embed processing metadata
  timestampFormat: 'seconds' | 'hms'; // timestamp representation
  wordTimestamps: boolean;     // word-level timing information
}

type OutputFormat = 'txt' | 'srt' | 'vtt' | 'json' | 'tsv';
```

### Processing Status
```typescript
interface ProcessingStatus {
  stage: 'loading' | 'preprocessing' | 'transcribing' | 'postprocessing' | 'complete';
  progress: number;            // completion percentage (0-100)
  currentSegment?: number;     // processing segment index
  estimatedTimeRemaining: number; // seconds remaining
  memoryUsage: number;         // current RAM usage
  processingSpeed: number;     // audio seconds per real second
}
```

## Constraints

### Hardware Requirements
- **Memory**: Model-dependent RAM requirements (1GB-8GB+)
- **CPU/GPU**: Processing power affects transcription speed
- **Storage**: Local model cache requires significant disk space
- **Audio Processing**: Sufficient compute for real-time or faster processing

### Audio Format Limitations
- **Supported Formats**: MP3, M4A, WAV, FLAC, and other common audio formats
- **Quality Requirements**: Better audio quality improves transcription accuracy
- **File Size**: Very large audio files may cause memory issues
- **Sample Rate**: Optimal performance with specific audio sampling rates

### Model Download Dependencies
- **Internet Connectivity**: Required for initial model downloads
- **Cache Storage**: Models stored in `~/.cache/whisper` directory
- **Download Time**: Large models require significant download time
- **Model Versions**: Different Whisper versions may have incompatible models

### Processing Performance Constraints
- **Real-time Factor**: Processing speed varies with model size and hardware
- **Memory Scaling**: Larger models and longer audio require more RAM
- **CPU Intensive**: Transcription without GPU acceleration is slower
- **Batch Limitations**: Limited concurrent processing capabilities

## Edge Cases

### Audio Quality and Content Issues
- **Poor Audio Quality**: Background noise, low volume, or distorted audio
- **Multiple Speakers**: Overlapping speech or rapid speaker changes
- **Accents and Dialects**: Non-standard pronunciation affecting accuracy
- **Technical Terminology**: Specialized vocabulary not in training data
- **Music and Effects**: Non-speech audio confusing transcription

### Model and Resource Problems
- **Model Download Failures**: Network issues preventing model acquisition
- **Cache Corruption**: Damaged model files requiring re-download
- **Insufficient Memory**: RAM exhaustion with large models or long audio
- **GPU Unavailability**: CUDA or GPU acceleration not available
- **Version Incompatibility**: Model versions not matching CLI tool version

### File and Format Complications
- **Unsupported Formats**: Audio formats not recognized by Whisper
- **Corrupted Files**: Damaged audio files causing processing failures
- **Permission Issues**: Unable to read input or write output files
- **Large File Processing**: Memory issues with very long audio recordings
- **Output Directory Problems**: Missing or inaccessible output directories

### Language and Translation Edge Cases
- **Language Misdetection**: Incorrect automatic language identification
- **Code-Switching**: Multiple languages within single audio file
- **Regional Variations**: Language variants not well-supported
- **Translation Accuracy**: Poor translation quality for specific language pairs
- **Rare Languages**: Limited support for less common languages

## Technical Debt

### Model Management Complexity
- Manual model selection without intelligent recommendations
- No automatic model cleanup or cache management
- Limited model performance profiling for hardware optimization
- No model update mechanism for improved versions

### Processing Pipeline Limitations
- No streaming or real-time transcription capabilities
- Basic error recovery for processing failures
- Limited preprocessing options for audio enhancement
- No batch processing optimization for multiple files

### Output Format Restrictions
- Fixed output formats without customization options
- Limited subtitle timing adjustment and formatting
- No integration with popular transcription workflow tools
- Basic metadata inclusion without comprehensive audio analysis

### Performance and Scalability Issues
- No GPU utilization optimization guidance
- Limited concurrent processing for batch operations
- Memory usage not optimized for resource-constrained environments
- No progress estimation or user feedback during long operations

## Dependencies

### Core Dependencies
- **OpenAI Whisper**: Primary speech-to-text engine
- **Python Runtime**: Programming language environment for Whisper
- **PyTorch**: Machine learning framework for model execution
- **Audio Processing Libraries**: FFmpeg or similar for audio format support

### System Dependencies
- **File System Access**: Read access to audio files, write access to output directory
- **Network Connectivity**: Required for initial model downloads
- **Memory Management**: Sufficient RAM for model loading and processing
- **CPU/GPU Resources**: Processing hardware for transcription computation

### Audio Processing Dependencies
- **FFmpeg**: Audio format conversion and preprocessing
- **Audio Codecs**: Support for various audio compression formats
- **Sample Rate Conversion**: Audio preprocessing for optimal model input
- **Noise Reduction**: Optional audio enhancement libraries

### Model Dependencies
- **Whisper Models**: Pre-trained neural network models for different sizes
- **Model Cache**: Local storage system for downloaded models
- **Version Management**: Model versioning and compatibility tracking
- **Download Infrastructure**: Reliable model distribution system

### Optional Dependencies
- **GPU Acceleration**: CUDA or similar for faster processing
- **Homebrew**: Package manager for macOS installation
- **Subtitle Tools**: Integration with subtitle editing and processing tools
- **Cloud Storage**: Backup and sharing of transcription results

### Development Dependencies
- **Python Package Management**: pip or similar for dependency installation
- **Testing Framework**: Audio transcription accuracy validation
- **Performance Profiling**: Processing speed and resource usage monitoring
- **Documentation**: Usage examples and best practices guides