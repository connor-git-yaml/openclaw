# Sag Skill Specification

## Intent

Provide high-quality text-to-speech capabilities through ElevenLabs API integration with macOS-style command-line interface. Enables natural voice synthesis, character voice generation, pronunciation control, and audio output management for accessibility, content creation, and interactive voice applications.

## Interface

### Core Operations
- **Text-to-Speech**: Convert text to natural-sounding speech with voice selection
- **Voice Management**: List, select, and configure available ElevenLabs voices
- **Audio Control**: Generate audio files or direct system playback
- **Voice Customization**: Apply delivery styles, emotions, and audio effects
- **Pronunciation Enhancement**: Text normalization and pronunciation control

### Command Structure
- `sag "TEXT"` - Basic text-to-speech with default voice
- `sag speak -v "VOICE_NAME" "TEXT"` - Voice-specific speech generation
- `sag voices` - List available voices
- `sag -o FILE.mp3 "TEXT"` - Audio file generation
- `sag prompting` - Display model-specific usage guidance

### Voice Enhancement Features
- **Audio Tags**: `[whispers]`, `[shouts]`, `[sings]`, `[laughs]` for v3 models
- **SSML Support**: `<break time="1.5s" />` for v2/v2.5 models with timing control
- **Normalization**: `--normalize auto|off` for numbers, URLs, and technical terms
- **Language Bias**: `--lang en|de|fr` for accent and pronunciation guidance

### Model Options
- **eleven_v3**: Expressive model with advanced emotional control (default)
- **eleven_multilingual_v2**: Stable multi-language support
- **eleven_flash_v2_5**: Fast generation with reduced latency

## Business Logic

### API Integration and Authentication
1. Authenticate with ElevenLabs API using secure API key management
2. Handle API rate limiting and quota management
3. Process voice synthesis requests with error handling and retries
4. Manage API response caching for efficiency

### Voice Synthesis Pipeline
1. Preprocess text for optimal speech synthesis quality
2. Apply voice-specific parameters and model selection
3. Submit synthesis requests to ElevenLabs API
4. Process audio responses for local playback or file storage

### Audio Processing and Playback
1. Handle various audio formats and quality settings
2. Integrate with macOS audio system for direct playback
3. Manage audio file generation and storage
4. Support streaming playback for long text content

### Voice Character and Style Management
1. Maintain voice catalog with metadata and capabilities
2. Apply emotional and delivery style tags appropriately
3. Handle pronunciation customization and text normalization
4. Support character voice presets for consistent user experience

### Text Enhancement and Preprocessing
1. Apply intelligent text normalization for technical content
2. Handle pronunciation challenges with respelling guidance
3. Process special characters, numbers, and URLs appropriately
4. Support multiple languages with proper accent application

## Data Structures

### Voice Configuration
```typescript
interface VoiceConfig {
  voiceId: string;             // ElevenLabs voice identifier
  name: string;                // voice display name
  description: string;         // voice characteristics
  category: string;            // voice category (male, female, etc.)
  language: string;            // primary language
  accent: string;              // accent specification
  previewUrl?: string;         // voice sample URL
  isCloned: boolean;           // custom vs predefined voice
  modelSupport: ModelSupport;  // compatible model versions
}

interface ModelSupport {
  v3: boolean;                 // eleven_v3 compatibility
  v2: boolean;                 // eleven_multilingual_v2 compatibility
  flash: boolean;              // eleven_flash_v2_5 compatibility
  ssml: boolean;               // SSML support availability
  audioTags: string[];         // supported audio enhancement tags
}
```

### Speech Request
```typescript
interface SpeechRequest {
  text: string;                // input text content
  voiceId: string;             // selected voice identifier
  model: 'eleven_v3' | 'eleven_multilingual_v2' | 'eleven_flash_v2_5';
  stability?: number;          // voice stability (0-1)
  similarityBoost?: number;    // similarity boost (0-1)
  style?: number;              // style exaggeration (0-1)
  speakerBoost?: boolean;      // speaker enhancement flag
  outputFormat: 'mp3' | 'wav' | 'pcm';
  outputPath?: string;         // file output location
  normalize: 'auto' | 'off';   // text normalization mode
  language?: string;           // language bias code
}
```

### Audio Enhancement
```typescript
interface AudioEnhancement {
  audioTags: AudioTag[];       // voice delivery modifiers
  ssmlElements: SSMLElement[]; // SSML markup elements
  pauseControls: PauseControl[]; // timing and pacing
  emotionalState: EmotionalState; // overall emotional context
}

interface AudioTag {
  type: 'whispers' | 'shouts' | 'sings' | 'laughs' | 'sighs' | 
        'sarcastic' | 'curious' | 'excited' | 'crying' | 'mischievously';
  position: number;            // character position in text
  scope: 'line' | 'phrase' | 'word'; // application scope
}

interface SSMLElement {
  type: 'break' | 'emphasis' | 'prosody';
  attributes: Record<string, string>; // element attributes
  content?: string;            // element content
  position: number;            // text position
}
```

### Generation Result
```typescript
interface GenerationResult {
  success: boolean;            // generation completion status
  audioUrl?: string;           // generated audio URL (for playback)
  audioPath?: string;          // local file path (for storage)
  duration: number;            // audio duration in seconds
  fileSize: number;            // audio file size in bytes
  quality: AudioQuality;       // generation quality metrics
  processingTime: number;      // API processing duration
  costEstimate?: number;       // usage cost estimate
}

interface AudioQuality {
  bitrate: number;             // audio bitrate
  sampleRate: number;          // audio sample rate
  channels: number;            // audio channel count
  format: string;              // audio format specification
}
```

### User Preferences
```typescript
interface UserPreferences {
  defaultVoice: string;        // preferred voice identifier
  defaultModel: string;        // preferred synthesis model
  outputFormat: string;        // preferred audio format
  outputDirectory: string;     // default file output location
  playbackDevice?: string;     // preferred audio output device
  normalizationMode: 'auto' | 'off'; // text normalization preference
  voiceSettings: VoiceSettings; // voice parameter defaults
}

interface VoiceSettings {
  stability: number;           // default stability setting
  similarityBoost: number;     // default similarity boost
  style: number;               // default style exaggeration
  speakerBoost: boolean;       // default speaker enhancement
}
```

## Constraints

### ElevenLabs API Limitations
- **API Key Required**: Valid ElevenLabs API key mandatory for all operations
- **Usage Quotas**: Character limits and monthly usage restrictions
- **Rate Limits**: Request frequency limitations per API tier
- **Model Availability**: Specific features limited to certain model versions

### Audio Processing Constraints
- **File Size Limits**: Generated audio files limited by storage and memory
- **Format Support**: Limited audio format options based on model capabilities
- **Quality Trade-offs**: Higher quality requires more processing time and bandwidth
- **Playback Dependencies**: System audio capabilities required for direct playback

### Text Processing Limitations
- **Character Limits**: Maximum text length per synthesis request
- **Language Support**: Limited language and accent combinations
- **SSML Complexity**: Advanced SSML features not supported in all models
- **Pronunciation Control**: Limited phonetic control compared to specialized tools

### System Dependencies
- **macOS Audio**: Requires macOS audio system for playback functionality
- **Network Connectivity**: Internet access required for ElevenLabs API
- **Storage Space**: Audio file generation requires sufficient disk space
- **Processing Power**: Real-time synthesis requires adequate system resources

## Edge Cases

### API and Service Issues
- **API Key Invalid**: Expired, restricted, or malformed authentication credentials
- **Quota Exceeded**: Monthly character or request limits reached
- **Service Outage**: ElevenLabs API temporarily unavailable
- **Rate Limiting**: Too many requests causing temporary blocks
- **Model Unavailable**: Requested voice model temporarily offline

### Audio Generation Problems
- **Large Text Processing**: Very long text causing generation timeouts
- **Special Characters**: Unicode or special characters causing synthesis issues
- **Pronunciation Failures**: Complex technical terms not pronouncing correctly
- **Audio Quality Degradation**: Poor network affecting audio generation quality
- **Memory Exhaustion**: Large audio generation consuming excessive system memory

### Voice and Model Complications
- **Voice Not Found**: Requested voice ID not available or removed
- **Model Incompatibility**: Voice not compatible with selected model
- **Custom Voice Issues**: Cloned voices not performing as expected
- **Language Mismatch**: Voice accent not matching content language
- **Style Conflicts**: Audio tags not compatible with selected voice

### System and Environment Issues
- **Audio Playback Failure**: System audio not available or configured incorrectly
- **File Permission Errors**: Unable to write audio files to specified locations
- **Disk Space Full**: Insufficient storage preventing audio file creation
- **Network Timeout**: Slow connections causing API request failures
- **Environment Variable Issues**: Missing or incorrect API key configuration

## Technical Debt

### Error Handling and Recovery
- Basic error reporting without specific guidance for different failure types
- Limited retry logic for transient API failures
- No graceful degradation when preferred voices or models unavailable
- Poor error context for debugging pronunciation and synthesis issues

### Audio Management Complexity
- Basic audio file organization without metadata or cataloging
- No streaming support for very long text content
- Limited audio format conversion capabilities
- Basic audio quality optimization without adaptive bitrate

### Voice and Character Management
- Manual voice selection without intelligent recommendations
- No voice similarity or clustering for character consistency
- Basic emotional state management without context awareness
- Limited voice training or customization feedback mechanisms

### Integration and Workflow Limitations
- Command-line only interface without graphical voice selection
- No integration with popular content creation or accessibility tools
- Basic scripting support without advanced automation features
- Limited batch processing for multiple text inputs

## Dependencies

### Core Dependencies
- **sag Binary**: Primary CLI tool for ElevenLabs TTS integration
- **ElevenLabs API**: Cloud text-to-speech service
- **HTTP Client**: Network communication for API requests
- **Audio Processing**: macOS audio system integration

### Authentication Dependencies
- **ElevenLabs API Key**: Valid authentication credential for service access
- **Environment Variables**: ELEVENLABS_API_KEY or SAG_API_KEY configuration
- **Secure Storage**: Safe credential storage and access management

### Audio Dependencies
- **macOS Audio Framework**: System audio playback and device management
- **Audio Codecs**: MP3, WAV, and other audio format support
- **File I/O**: Audio file reading, writing, and streaming capabilities
- **Audio Device Management**: Speaker and audio output device control

### Text Processing Dependencies
- **Unicode Support**: International character handling and encoding
- **Language Detection**: Text language identification for proper synthesis
- **Normalization Libraries**: Number, URL, and abbreviation processing
- **Markup Processing**: SSML and audio tag parsing and application

### System Dependencies
- **Network Stack**: HTTPS connectivity for API communication
- **File System**: Audio file storage and management
- **Process Management**: Background audio generation and playback
- **Configuration Storage**: User preferences and voice settings persistence

### Optional Dependencies
- **Homebrew**: Package manager for macOS installation (steipete/tap)
- **Audio Editing Tools**: Post-processing and audio enhancement
- **Integration APIs**: Webhook or API integration for external applications
- **Monitoring**: Usage tracking and performance monitoring capabilities