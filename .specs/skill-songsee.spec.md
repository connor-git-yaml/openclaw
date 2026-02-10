# Songsee Skill Specification

## Intent

Generate spectrograms and audio feature visualizations from audio files using the songsee CLI tool. Enables audio analysis, frequency visualization, and acoustic feature extraction for music analysis and audio processing workflows.

## Interface

### Core Operations
- **Spectrogram Generation**: Create visual frequency-time representations of audio
- **Feature Panel Creation**: Generate comprehensive audio feature visualizations
- **Audio Analysis**: Extract and visualize acoustic characteristics
- **Batch Processing**: Process multiple audio files for comparison analysis

### CLI Structure
- **Basic Spectrogram**: `songsee input.mp3 --output spectrogram.png`
- **Feature Panel**: `songsee input.wav --features --output analysis.png`
- **Custom Parameters**: `songsee audio.m4a --fft-size 2048 --window hamming`
- **Batch Mode**: `songsee *.mp3 --batch --output-dir visualizations/`

## Business Logic

### Audio Processing and Analysis
1. Load and validate audio files in various formats
2. Apply FFT and windowing functions for frequency analysis
3. Generate time-frequency representations and spectrograms
4. Extract acoustic features like MFCCs, spectral centroid, and energy

### Visualization Generation
1. Create high-quality spectrogram images with configurable parameters
2. Generate comprehensive feature panel layouts
3. Apply color mappings and scaling for visual clarity
4. Export visualizations in multiple image formats

### Batch Processing and Comparison
1. Process multiple audio files with consistent parameters
2. Generate comparative visualizations across multiple tracks
3. Create summary reports and analysis overviews
4. Handle large audio collections efficiently

## Data Structures

### Audio Input
```typescript
interface AudioFile {
  path: string; // audio file path
  format: string; // audio format (mp3, wav, m4a, etc.)
  duration: number; // audio length in seconds
  sampleRate: number; // sample rate in Hz
  channels: number; // number of audio channels
  bitrate?: number; // audio bitrate
}
```

### Analysis Parameters
```typescript
interface AnalysisConfig {
  fftSize: number; // FFT window size
  hopSize: number; // hop length between windows
  windowType: 'hamming' | 'hanning' | 'blackman'; // windowing function
  frequencyRange: [number, number]; // min/max frequency
  timeRange?: [number, number]; // analysis time window
  colorMap: string; // visualization color scheme
}
```

### Visualization Output
```typescript
interface Visualization {
  type: 'spectrogram' | 'features' | 'comparison';
  outputPath: string; // generated image path
  format: 'png' | 'jpg' | 'svg'; // image format
  dimensions: { width: number; height: number };
  metadata: VisualizationMetadata; // analysis information
}

interface VisualizationMetadata {
  audioSource: string; // source audio file
  analysisParams: AnalysisConfig; // processing parameters
  features?: FeatureData; // extracted features
  timestamp: Date; // generation time
}
```

### Feature Extraction
```typescript
interface FeatureData {
  spectralCentroid: number[]; // spectral centroid over time
  spectralRolloff: number[]; // spectral rolloff frequencies
  mfcc: number[][]; // mel-frequency cepstral coefficients
  energy: number[]; // RMS energy over time
  zeroCrossingRate: number[]; // zero crossing rate
  tempo?: number; // estimated tempo (BPM)
}
```

## Constraints

### Audio Format Support
- **Supported Formats**: Limited to formats supported by underlying audio libraries
- **Sample Rates**: Performance varies with different sample rates
- **Bit Depths**: May have limitations on high-resolution audio
- **File Size**: Very large audio files may cause memory issues

### Processing Performance
- **CPU Intensive**: FFT operations require significant processing power
- **Memory Usage**: Large audio files and high FFT sizes increase memory needs
- **Generation Time**: Complex visualizations take considerable processing time
- **Concurrent Processing**: Limited parallelization for batch operations

### Output Quality and Configuration
- **Resolution Limits**: Image output resolution constraints
- **Color Accuracy**: Limited color palette options for visualizations
- **Parameter Sensitivity**: Small parameter changes can significantly affect output
- **File Size**: High-quality visualizations create large image files

## Edge Cases

### Audio Processing Issues
- **Corrupted Audio**: Damaged or invalid audio files
- **Unsupported Codecs**: Audio formats not supported by system libraries
- **Silent Audio**: Audio files with no signal or very low levels
- **Clipped Audio**: Distorted audio affecting analysis accuracy
- **Variable Sample Rates**: Audio with changing sample rates

### Generation Failures
- **Memory Exhaustion**: Insufficient RAM for large audio analysis
- **Disk Space**: Insufficient storage for output images
- **Processing Timeout**: Very long audio files causing timeouts
- **Invalid Parameters**: FFT or visualization parameters causing errors
- **Permission Issues**: Cannot write to specified output directories

### Batch Processing Problems
- **Mixed Formats**: Different audio formats in batch requiring format handling
- **Naming Conflicts**: Output filename collisions in batch processing
- **Partial Failures**: Some files failing in batch while others succeed
- **Resource Exhaustion**: Running out of system resources during batch processing

## Technical Debt

### Limited Customization Options
- Basic visualization templates without advanced customization
- Limited color scheme and styling options
- No interactive visualization capabilities
- Minimal annotation and labeling features

### Performance Optimization
- No caching of intermediate processing results
- Limited multi-threading support for batch operations
- No progressive processing for very large files
- Basic memory management without optimization

### Integration Limitations
- Command-line only interface without programmatic API
- Limited metadata extraction and preservation
- No integration with audio editing or analysis workflows
- Minimal export format options

## Dependencies

### Core Dependencies
- **songsee CLI**: Songsee command-line tool installation
- **Audio Libraries**: System audio processing libraries (FFmpeg, etc.)
- **Image Processing**: Graphics libraries for visualization generation
- **Mathematical Libraries**: FFT and signal processing implementations

### System Dependencies
- **Operating System**: Platform compatibility for audio and graphics
- **Audio Codecs**: System codecs for various audio format support
- **Graphics Output**: Image format support and graphics drivers
- **File System**: Storage access for input audio and output images

### Optional Dependencies
- **Batch Processing**: Shell scripting tools for automation
- **Format Conversion**: Additional audio format conversion utilities
- **Advanced Analysis**: Machine learning libraries for feature extraction
- **Visualization Tools**: Integration with plotting and analysis software