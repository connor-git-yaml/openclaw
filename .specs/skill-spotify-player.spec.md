# Spotify Player Skill Specification

## Intent

Provide terminal-based Spotify playback control and music search through spogo or spotify_player CLI tools. Enables music discovery, playlist management, and playback automation without requiring GUI applications.

## Interface

### Core Operations
- **Playback Control**: Play, pause, skip tracks and control volume
- **Music Search**: Search tracks, albums, artists, and playlists
- **Queue Management**: Add songs to queue and manage playback order  
- **Device Control**: Select and control Spotify Connect devices
- **Library Access**: Browse user's saved music and playlists

### CLI Options
- **spogo** (preferred): Lightweight Spotify CLI client
- **spotify_player**: Full-featured terminal Spotify client

## Business Logic

### Authentication and API Integration
1. Handle Spotify API authentication with OAuth tokens
2. Manage client credentials and user authorization
3. Process API rate limiting and request management
4. Handle token refresh and session persistence

### Music Discovery and Search
1. Execute search queries across Spotify's catalog
2. Parse and display search results with metadata
3. Handle track, album, artist, and playlist searches
4. Process search filtering and result ranking

### Playback Control and Queue Management
1. Send playback commands to active Spotify devices
2. Manage playback queue and track ordering
3. Handle volume control and audio settings
4. Process shuffle, repeat, and crossfade options

## Data Structures

### Track Information
```typescript
interface SpotifyTrack {
  id: string; // Spotify track ID
  name: string; // track title
  artists: Artist[]; // track artists
  album: Album; // album information
  duration: number; // track length (ms)
  popularity: number; // track popularity score
  uri: string; // Spotify URI
}

interface Artist {
  id: string; name: string; uri: string;
}

interface Album {
  id: string; name: string; images: Image[];
}
```

### Playback State
```typescript
interface PlaybackState {
  isPlaying: boolean; // playback status
  track: SpotifyTrack; // current track
  position: number; // playback position (ms)
  volume: number; // volume level (0-100)
  device: Device; // active device
  shuffleState: boolean; // shuffle enabled
  repeatState: 'off' | 'context' | 'track'; // repeat mode
}

interface Device {
  id: string; name: string; type: string; 
  isActive: boolean; volumePercent: number;
}
```

## Constraints

### Spotify API Limitations
- **Premium Account**: Full playback control requires Spotify Premium
- **Rate Limiting**: API request frequency restrictions
- **Regional Restrictions**: Content availability varies by location
- **Device Requirements**: Active Spotify device needed for control

### Authentication Requirements
- **API Credentials**: Spotify app registration and client secrets
- **User Authorization**: OAuth flow for user permission
- **Token Management**: Access token expiration and refresh

## Edge Cases

### Authentication Issues
- **Token Expiry**: Access tokens becoming invalid
- **Authorization Revoked**: User revoking app permissions
- **API Limitations**: Rate limits or service unavailability
- **Account Issues**: Spotify account suspension or restrictions

### Playback Problems
- **No Active Device**: No Spotify device available for control
- **Premium Required**: Attempting control without Premium subscription
- **Content Unavailable**: Tracks not available in user's region
- **Device Offline**: Selected device not reachable

## Technical Debt

### Multiple CLI Options
- Supporting two different CLI tools adds complexity
- Inconsistent interfaces between spogo and spotify_player
- Different feature sets and capabilities

### Configuration Management
- Manual setup of Spotify API credentials
- Complex OAuth flow for initial authentication
- No unified configuration management

## Dependencies

### Core Dependencies
- **spogo** or **spotify_player**: Terminal Spotify clients
- **Spotify API**: Spotify Web API access
- **OAuth Libraries**: Authentication flow handling
- **HTTP Client**: API request capabilities

### System Dependencies
- **Network Connectivity**: Internet access for Spotify API
- **Terminal Environment**: Command-line interface support
- **Audio System**: System audio for local playback (optional)
- **Package Manager**: Installation tool support

### Authentication Dependencies
- **Spotify Account**: Valid Spotify user account
- **API Registration**: Spotify app credentials
- **Premium Subscription**: For full playback control features
- **Device Authorization**: Authorized Spotify Connect devices