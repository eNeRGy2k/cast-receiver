# KoreStream Ultra Receiver

A custom Google Cast receiver application for media streaming with advanced DRM support and manifest optimization.

**Version:** v1.1.3

## Overview

KoreStream Ultra Receiver is an HTML5-based Cast receiver application designed to handle complex streaming scenarios, including:

- **DASH & HLS streaming** with adaptive bitrate playback
- **Multiple DRM systems** (Widevine, ClearKey, and others)
- **Dynamic manifest manipulation** and optimization
- **Audio codec filtering** and compatibility management
- **Live streaming** with improved timing synchronization
- **NVIDIA Shield compatibility** with proper state cleanup

## Features

### 🎬 Streaming Protocol Support
- **DASH (Dynamic Adaptive Streaming over HTTP)** - Full MPD manifest support
- **HLS (HTTP Live Streaming)** - M3U8 playlist support
- **Clear Key DRM** - For non-encrypted or standard encryption workflows
- **Widevine DRM** - Premium video content protection

### 🛠 Advanced Capabilities

#### Manifest Processing
- Automatic manifest parsing and modification
- Availability Start Time (AST) correction for live streams
- Dynamic buffer configuration
- Segment timeline normalization
- STPP subtitle track removal

#### DRM Management
- Clear Key integration with automatic JWKS formatting
- Widevine license server configuration
- Selective DRM system filtering based on stream type
- Protection scheme validation and cleanup

#### Audio Processing
- AC-3 and E-AC-3 codec filtering
- Audio channel count preferences
- Language normalization (defaults to Spanish)
- Multi-codec preference configuration

#### Reliability Features
- Automatic error loop detection
- State reset on new load requests
- Network retry configuration
- UTC timing removal for live stream desynchronization prevention

## Technical Details

### Architecture

The receiver leverages the **Google Cast Receiver Framework v3** and **Shaka Player** for media playback:

```
┌─────────────────────────────────────────┐
│   Cast Receiver Framework               │
│   ├── Media Player                      │
│   └── Custom Message Channel            │
├─────────────────────────────────────────┤
│   Playback Configuration                │
│   ├── Manifest Request Handler          │
│   ├── Segment Request Handler           │
│   └── Media Playback Info Handler       │
├─────────────────────────────────────────┤
│   Shaka Player Engine                   │
│   ├── DASH/HLS Decoder                  │
│   ├── DRM Manager                       │
│   └── Networking Engine                 │
└─────────────────────────────────────────┘
```

### Key Components

#### 1. **Manifest Request Handler**
- Intercepts DASH MPD and HLS manifests
- Performs XML-based modifications
- Injects DRM configurations
- Optimizes buffer parameters

#### 2. **Segment Request Handler**
- Adds custom headers to segment requests
- Manages authentication tokens
- Handles credential configuration

#### 3. **Media Playback Info Handler**
- Extracts custom data from load requests
- Configures DRM parameters dynamically
- Routes between DASH and HLS pipelines

#### 4. **Event Listeners**
- **PLAYER_LOAD_COMPLETE**: Applies audio track filtering, configures response filters
- **ERROR**: Detects unrecoverable errors and notifies sender
- **ENDED**: Handles error loop detection for static content

### Custom Data Format

The sender application can pass custom metadata via the `media.customData` field:

```javascript
{
  headers: {
    "Authorization": "Bearer token",
    "User-Agent": "Custom-Agent"
  },
  clearKeys: {
    "hexKeyId1": "hexKeyValue1",
    "hexKeyId2": "hexKeyValue2"
  },
  licenseUrl: "https://license-server.example.com/widevine"
}
```

## Playback Configuration

### Shaka Player Settings

```javascript
{
  abr: { enabled: false },                    // Manual bitrate selection
  streaming: {
    jumpLargeGaps: true,                      // Skip discontinuities
    liveIgnoreNtp: true,                      // Ignore NTP in live streams
    bufferingGoal: 4,                         // Seconds
    rebufferingGoal: 2,                       // Seconds
    stallEnabled: true,                       // Detect playback stalls
    retryParameters: { maxAttempts: 2, timeout: 5000 }
  },
  manifest: {
    hls: { ignoreTextStreamFailures: true, allowInsecureScheme: true },
    dash: { ignoreMinBufferTime: true, autoCorrectDrift: true },
    retryParameters: { maxAttempts: 1, timeout: 4000 }
  },
  drm: {
    retryParameters: { maxAttempts: 0 }       // No retry for DRM
  },
  preferredAudioCodecs: ['mp4a.40.2', 'mp4a', 'aac'],
  preferredVideoCodecs: ['avc1', 'h264']
}
```

## Installation & Deployment

### Requirements
- Google Cast device (Chromecast, Smart TV, Android TV, etc.)
- Cast receiver registration on Google Cast Console
- HTTPS hosting (Cast requires secure contexts)

### Setup Steps

1. **Create Cast Application**
   - Register your app on [Google Cast SDK Console](https://cast.google.com/publish)
   - Get your application ID

2. **Deploy HTML File**
   - Upload `index.html` to your HTTPS-enabled web server
   - Ensure proper cache control headers are set

3. **Point Cast Application**
   - Configure your Cast app to load this HTML file
   - Use custom channel: `urn:x-cast:com.equisde.korestream`

4. **Sender Application**
   - Implement Cast sender library in your application
   - Load media with custom data parameters

## Error Handling

The receiver implements several error handling strategies:

### Recoverable Errors
- Network timeouts → Automatic retry with backoff
- Manifest parsing issues → Logged and ignored
- Audio track unavailability → Fallback to available tracks

### Unrecoverable Errors
- Shaka error codes: 1001, 6012, 6007, 6000
- Action: Notify sender and stop playback for immediate skip

## NVIDIA Shield Compatibility

Special handling for NVIDIA Shield devices:
- Complete state cleanup on each new load request
- Video element attribute reset
- Active custom data clearing
- Error loop flag reset

This prevents playback state corruption from previous streams.

## Browser Compatibility

- **Chrome/Chromium-based devices** (Full support)
- **Android devices** running Chromecast app (Full support)
- **Smart TVs** with Cast support (Full support)
- **NVIDIA Shield** (Verified compatible)

## Debugging

### Console Logging
The receiver logs to the browser console with prefixes:
- `[KoreStream]` - General receiver logs
- `[CustomChannel]` - Custom message channel events
- `[Manifest Parsing Error]` - Manifest processing issues
- `[Audio Track Filter Warning]` - Audio selection warnings
- `[JWKS Error]` - DRM key formatting issues
- `[Fail-Fast]` - Unrecoverable error detection

### Inspecting Player State
Use the Cast receiver debug interface to:
- Monitor active playback configuration
- Review sender messages
- Inspect error events
- Check network activity

## Version History

### v1.1.3 (Current)
- Enhanced NVIDIA Shield compatibility
- Improved manifest parsing resilience
- Better audio codec filtering
- DRM system selection refinement

## License

Not specified. Check with repository owner for licensing terms.

## Support

For issues, questions, or contributions, please open an issue in the repository.

## Custom Channel Communication

### Message Format

**From Sender to Receiver (Custom Channel):**
```javascript
{
  type: 'LOAD',
  media: {
    contentId: 'stream-url',
    contentType: 'application/dash+xml',
    customData: { /* config */ }
  }
}
```

**From Receiver to Sender (Error Notification):**
```javascript
{
  type: 'FATAL_ERROR',
  reason: 'error_message',
  code: error_code_number
}
```

---

**Repository:** https://github.com/eNeRGy2k/cast-receiver  
**Language:** HTML5/JavaScript  
**Framework:** Google Cast Receiver Framework v3 + Shaka Player
