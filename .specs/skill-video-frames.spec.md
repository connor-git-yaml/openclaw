# Video Frames Skill Specification

## Intent

Extract individual frames or short video clips from video files using FFmpeg for video analysis, thumbnail generation, and content extraction workflows.

## Interface

### Core Operations
- **Frame Extraction**: Extract single or multiple frames at specific timestamps
- **Clip Generation**: Create short video segments from longer videos
- **Thumbnail Creation**: Generate preview images from video content
- **Batch Processing**: Process multiple videos or time ranges efficiently

### FFmpeg Integration
- **Time-based Extraction**: Extract frames at specific timestamps
- **Interval Sampling**: Sample frames at regular intervals
- **Quality Control**: Configure output format and quality settings
- **Format Support**: Handle various input video formats and codecs

## Business Logic

### Frame Extraction and Processing
1. Parse video metadata to determine duration and properties
2. Calculate frame positions based on timestamps or intervals
3. Execute FFmpeg commands for frame extraction
4. Handle output file naming and organization

### Video Analysis and Segmentation
1. Create video clips from specified time ranges
2. Generate thumbnails for video preview and navigation
3. Process batch operations across multiple video files
4. Handle video format conversion during extraction

## Data Structures

### Video Input
```typescript
interface VideoFile {
  path: string; // video file path
  format: string; // video container format
  duration: number; // video length in seconds
  resolution: { width: number; height: number };
  frameRate: number; // frames per second
  codecs: { video: string; audio?: string };
}
```

### Extraction Configuration
```typescript
interface ExtractionConfig {
  timestamps: number[]; // specific time points (seconds)
  interval?: number; // regular interval sampling
  outputFormat: 'png' | 'jpg' | 'bmp'; // image format
  quality?: number; // output quality (1-31 for jpg)
  scale?: string; // resolution scaling
  outputDir: string; // output directory
}
```

### Frame Output
```typescript
interface ExtractedFrame {
  timestamp: number; // source timestamp
  filename: string; // output filename
  path: string; // full file path
  size: number; // file size in bytes
  dimensions: { width: number; height: number };
}
```

## Constraints

### FFmpeg Dependencies
- **FFmpeg Required**: FFmpeg must be installed and available
- **Codec Support**: Limited by FFmpeg's codec availability
- **Performance**: Processing speed depends on video resolution and length
- **Memory Usage**: Large videos may require significant memory

### Format and Quality Limitations
- **Input Formats**: Support limited to FFmpeg-compatible formats
- **Output Quality**: Quality vs file size tradeoffs
- **Resolution Limits**: Maximum output resolution constraints
- **Color Accuracy**: Color space conversion limitations

## Edge Cases

### Video Processing Issues
- **Corrupted Videos**: Damaged or incomplete video files
- **Unsupported Codecs**: Video codecs not available in FFmpeg build
- **Invalid Timestamps**: Extraction times beyond video duration
- **Large Files**: Memory or disk space exhaustion during processing

### Output Problems
- **Write Permissions**: Cannot create output files or directories
- **Disk Space**: Insufficient storage for extracted frames
- **Filename Conflicts**: Output filename collisions
- **Format Errors**: Image format encoding failures

## Technical Debt

### Limited Customization
- Basic FFmpeg parameter exposure without advanced options
- No built-in video analysis or intelligent frame selection
- Limited batch processing optimization

### Error Handling
- Basic error reporting without detailed troubleshooting
- No recovery mechanisms for partial processing failures
- Limited validation of extraction parameters

## Dependencies

### Core Dependencies
- **FFmpeg**: Video processing and frame extraction engine
- **File System**: Storage access for input videos and output frames
- **Shell Environment**: Command execution capabilities

### System Dependencies
- **Codec Libraries**: Video and image codec support
- **Storage Space**: Adequate disk space for frame output
- **Processing Power**: CPU for video decoding and processing