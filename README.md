# KoreStream Ultra Receiver

A custom Google Cast receiver application for media streaming with advanced DRM support and manifest optimization. **Production-grade with critical hardware safety guarantees.**

**Version:** v1.2.0-SAFE

## Overview

KoreStream Ultra Receiver is an HTML5-based Cast receiver application designed to handle complex streaming scenarios safely across all Cast device types, from Nest Hub to NVIDIA Shield:

- **DASH & HLS streaming** with adaptive bitrate playback
- **Multiple DRM systems** (Widevine, ClearKey, and others)
- **Dynamic manifest manipulation** with hardware safety guards
- **Device-aware codec filtering** preventing kernel-level panics
- **Live streaming** with standards-compliant detection (DASH MPD@type)
- **NVIDIA Shield optimization** with proper state cleanup
- **Comprehensive error handling** with device-specific fallbacks

## What's New in v1.2.0-SAFE

This is a **critical security release** addressing hardware-level vulnerabilities identified during streaming architecture review:

### 🔴 Critical Fixes

1. **AC-3/E-AC-3 Audio Codec Safety**
   - ❌ v1.1.3: Attempted "graceful fallback" could cause kernel panic
   - ✅ v1.2.0-SAFE: Device-specific purge - mandatory removal in Nest Hub/Chromecast v3
   - Prevents: Audio decoder freezing, screen lock, receiver crash

2. **STPP Subtitle Pipeline Protection**
   - ❌ v1.1.3: Passthrough STPP, assume graceful degradation
   - ✅ v1.2.0-SAFE: Runtime Shaka version detection, conditional removal
   - Prevents: Subtitle pipeline breakage in Chromecast v3 + Shaka < 3.2

3. **Live Stream Detection via Standards**
   - ❌ v1.1.3: URL pattern matching (fragile, false positives)
   - ✅ v1.2.0-SAFE: Parse DASH manifest `MPD@type="dynamic"` (ISO/IEC 23009-1)
   - Result: Accurate buffer tuning, proper segment prefetching

4. **Dual Codec Capability Validation**
   - ❌ v1.1.3: User-agent only, no capability verification
   - ✅ v1.2.0-SAFE: User-agent + `MediaSource.isTypeSupported()` dual validation
   - Result: No false codec declarations, clean error messages

### 📊 Compatibility Matrix

| Device | v1.1.3 | v1.2.0-SAFE | Notes |
|--------|--------|-------------|-------|
| **Nest Hub** | ⚠️ Risky AC-3 | ✅ Safe (AC-3 purged) | Low memory, limited codec support |
| **Chromecast v3** | ⚠️ STPP crash risk | ✅ Safe (version-aware) | Old Shaka, needs protection |
| **Chromecast Ultra** | ✅ OK | ✅ Enhanced | VP9 + H.264 support |
| **Android TV** | ✅ OK | ✅ Enhanced | Variable codec support |
| **NVIDIA Shield TV** | ✅ OK | ✅ Optimized | Full codec support, AC-3 allowed |

## Features

### 🎬 Streaming Protocol Support
- **DASH (Dynamic Adaptive Streaming over HTTP)** - Full MPD manifest support with standards-compliant parsing
- **HLS (HTTP Live Streaming)** - M3U8 playlist support
- **Clear Key DRM** - For non-encrypted or standard encryption workflows
- **Widevine DRM** - Premium video content protection with fallback handling

### 🛠 Advanced Capabilities

#### Manifest Processing (v1.2.0-SAFE)
- Automatic manifest parsing and modification
- Availability Start Time (AST) correction for live streams
- Dynamic buffer configuration (VOD: 10s/4s, Live: 3s/1.5s)
- Segment timeline normalization
- **STPP subtitle handling**: Version-aware, safe removal in old Shaka
- **Stream type detection**: Parse `MPD@type` attribute with URL fallback

#### Device Detection (v1.2.0-SAFE)
- Runtime device identification (Nest Hub, Shield, Chromecast, Android TV)
- Device-specific codec preferences
- User-agent + `MediaSource.isTypeSupported()` dual validation
- Automatic capability negotiation

#### Audio Processing (v1.2.0-SAFE)
- **Safe AC-3/E-AC-3 handling**: 
  - Nest Hub/Chromecast v3: Mandatory purge (kernel safety)
  - NVIDIA Shield: Allowed (safe hardware decode)
  - Clear logging of decisions
- Audio channel count preferences
- Language normalization (defaults to Spanish)
- Multi-codec preference configuration with fallback

#### DRM Management
- Clear Key integration with automatic JWKS formatting
- Widevine license server configuration
- Selective DRM system filtering based on stream type
- Protection scheme validation and cleanup
- Improved retry parameters (15s timeout for license servers)

#### Reliability Features
- Automatic error loop detection
- State reset on new load requests
- Enhanced network retry configuration:
  - Manifest: 5 attempts, 10s timeout
  - Segment: 3 attempts, 8s timeout
  - DRM License: 2 attempts, 15s timeout
- UTC timing removal for live stream desynchronization prevention

## Technical Details

### Architecture

The receiver leverages the **Google Cast Receiver Framework v3** and **Shaka Player** for media playback:

```
┌────────────────────────────────────────────────┐
│   Cast Receiver Framework v3                   │
│   ├── Media Player                             │
│   └── Custom Message Channel                   │
├────────────────────────────────────────────────┤
│   Device Detection & Capability Validation     │
│   ├── User-Agent Parsing                       │
│   └── MediaSource.isTypeSupported()            │
├────────────────────────────────────────────────┤
│   Playback Configuration (Device-Aware)        │
│   ├── Manifest Request Handler                 │
│   ├── Segment Request Handler                  │
│   └── Media Playback Info Handler              │
├────────────────────────────────────────────────┤
│   Manifest Processing (Hardware-Safe)          │
│   ├── DASH MPD Parsing & Modification          │
│   ├── Stream Type Detection (MPD@type)         │
│   ├── Device-Specific Audio Filtering          │
│   └── Shaka Version-Aware Subtitle Handling    │
├────────────────────────────────────────────────┤
│   Shaka Player Engine                          │
│   ├── DASH/HLS Decoder                         │
│   ├── DRM Manager                              │
│   └── Networking Engine                        │
└────────────────────────────────────────────────┘
```

### Key Components

#### 1. **Device Detection Module** (NEW in v1.2.0-SAFE)
```javascript
detectDevice()           // User-agent parsing (Nest Hub, Shield, etc.)
validateCodecsWithMSE()  // Dual validation against MediaSource API
```
- Identifies device capabilities
- Prevents false codec declarations
- Enables device-specific optimizations

#### 2. **Stream Type Detection** (FIXED in v1.2.0-SAFE)
```javascript
detectStreamTypeFromManifest()  // Parse MPD@type="dynamic"
```
- Authoritative DASH standard compliance
- Fallback to URL heuristic if absent
- Enables accurate buffer tuning

#### 3. **Manifest Request Handler** (HARDENED in v1.2.0-SAFE)
- Intercepts DASH MPD and HLS manifests
- Performs XML-based modifications
- Device-specific audio codec filtering (Nest Hub/Shield differential)
- Runtime Shaka version detection for subtitle safety
- Injects DRM configurations
- Optimizes buffer parameters based on stream type

#### 4. **Segment Request Handler**
- Adds custom headers to segment requests
- Manages authentication tokens
- Handles credential configuration

#### 5. **Media Playback Info Handler** (ENHANCED in v1.2.0-SAFE)
- Extracts custom data from load requests
- Configures DRM parameters dynamically
- Routes between DASH and HLS pipelines
- Applies device-specific codec preferences
- Dynamic buffer configuration (VOD vs Live)

#### 6. **Event Listeners**
- **PLAYER_LOAD_COMPLETE**: Applies audio track filtering with safety checks, configures response filters
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

### Shaka Player Settings (v1.2.0-SAFE)

```javascript
{
  abr: { 
    enabled: false,  // Conservative by default
    defaultBandwidthEstimate: 5000000  // 5 Mbps fallback
  },
  
  streaming: {
    jumpLargeGaps: true,
    liveIgnoreNtp: true,
    bufferingGoal: 4,      // VOD: 10s, Live: 3s (dynamic)
    rebufferingGoal: 2,    // VOD: 4s, Live: 1.5s (dynamic)
    stallEnabled: true,
    preferLiveSegments: false,  // Dynamic based on stream type
    retryParameters: { maxAttempts: 3, timeout: 8000 }
  },
  
  manifest: {
    hls: { ignoreTextStreamFailures: true, allowInsecureScheme: true },
    dash: { ignoreMinBufferTime: true, autoCorrectDrift: true },
    retryParameters: { maxAttempts: 5, timeout: 10000 }  // Enhanced retry
  },
  
  drm: {
    retryParameters: { maxAttempts: 2, timeout: 15000 }  // DRM now has retry
  },
  
  // Device-specific codec preferences (validated via MediaSource)
  preferredAudioCodecs: ['mp4a.40.2', 'mp4a.40.5', 'mp4a', 'aac'],
  preferredVideoCodecs: ['vp9', 'avc1', 'h264', 'avc3'],  // Dynamic per device
  
  preferredAudioChannelCount: 2,
  preferFewerChannels: true
}
```

## Installation & Deployment

### Requirements
- Google Cast device (Chromecast, Smart TV, Android TV, Nest Hub, NVIDIA Shield, etc.)
- Cast receiver registration on Google Cast Console
- HTTPS hosting (Cast requires secure contexts)
- For v1.2.0-SAFE: Verify device compatibility via testing

### Setup Steps

1. **Create Cast Application**
   - Register your app on [Google Cast SDK Console](https://cast.google.com/publish)
   - Get your application ID

2. **Deploy HTML File**
   - Upload `index.html` (v1.2.0-SAFE) to your HTTPS-enabled web server
   - Ensure proper cache control headers are set
   - Test on target device types before production

3. **Point Cast Application**
   - Configure your Cast app to load this HTML file
   - Use custom channel: `urn:x-cast:com.equisde.korestream`

4. **Sender Application**
   - Implement Cast sender library in your application
   - Load media with custom data parameters
   - Monitor console logs for [DEVICE], [CODEC-VALIDATION], [STREAM-TYPE] messages

## Error Handling

The receiver implements several error handling strategies:

### Recoverable Errors
- Network timeouts → Automatic retry with backoff (device-aware)
- Manifest parsing issues → Logged and ignored
- Audio codec unavailability → Fallback to available codecs per device
- STPP subtitle failure → Graceful degradation in Shaka 3.2+

### Unrecoverable Errors
- Shaka error codes: 1001, 6012, 6007, 6000
- Action: Notify sender and stop playback for immediate skip

### Device-Specific Safety
- **Nest Hub**: AC-3 mandatory removal prevents kernel panic
- **Chromecast v3**: STPP removed if Shaka < 3.2 prevents pipeline breakage
- **NVIDIA Shield**: Full codec support with safety validation

## Debugging

### Console Logging (v1.2.0-SAFE Enhanced)
The receiver logs to the browser console with detailed prefixes:
- `[KoreStream]` - General receiver logs and initialization
- `[DEVICE]` - Device detection results
- `[CODEC-VALIDATION]` - Codec support validation (via MediaSource)
- `[CODEC]` - Codec selection decisions
- `[AUDIO]` - Audio codec filtering and fallback decisions
- `[SUBTITLE]` - Subtitle handling (STPP version check, removal decisions)
- `[STREAM-TYPE]` - Stream type detection (LIVE vs VOD, method used)
- `[BUFFER]` - Buffer configuration per stream type
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
- Verify device detection and capability validation

## Version History

### v1.2.0-SAFE (Current) - PRODUCTION RELEASE
**Critical Security & Compatibility Release**
- ✅ AC-3/E-AC-3 device-specific safety (Nest Hub/Chromecast v3 kernel panic prevention)
- ✅ STPP subtitle version-aware handling (Shaka < 3.2 protection)
- ✅ Standards-compliant stream type detection (MPD@type parsing)
- ✅ Dual codec capability validation (User-Agent + MediaSource API)
- ✅ Enhanced retry parameters (manifests, segments, DRM)
- ✅ Dynamic buffer configuration (VOD vs Live)
- ✅ Device-aware logging with safety decision tracking
- ⚠️ **Not backward compatible with v1.1.3 deployment** - Review device compatibility before upgrading

### v1.1.3 (Previous)
- Enhanced NVIDIA Shield compatibility
- Improved manifest parsing resilience
- Basic audio codec filtering (⚠️ unsafe AC-3 handling)
- DRM system selection refinement
- ❌ **DEPRECATED** - Security issues identified, upgrade to v1.2.0-SAFE

## Migration Guide: v1.1.3 → v1.2.0-SAFE

**Testing Requirements (MANDATORY)**
1. Test on Nest Hub: Verify AC-3 streams fallback to AAC without crash
2. Test on Chromecast v3: Verify STPP removal prevents pipeline breakage (if Shaka < 3.2)
3. Test on NVIDIA Shield: Verify AC-3 allowed and AC-3 streams play correctly
4. Test DASH with `MPD@type="dynamic"`: Verify live stream detection
5. Test DASH with `MPD@type="static"`: Verify VOD detection
6. Monitor console logs for `[DEVICE]`, `[CODEC-VALIDATION]`, `[AUDIO]` messages

**Breaking Changes**
- Nest Hub: AC-3 audio tracks will be silently filtered (no audio output for AC-3-only streams)
- Chromecast v3 + old Shaka: STPP subtitles will be silently removed
- Audio codec preference order changed: AAC prioritized, AC-3 fallback only on Shield

**Non-Breaking Enhancements**
- All DRM functionality preserved (ClearKey, Widevine)
- NVIDIA Shield optimizations enhanced
- Error handling improved
- Logging enhanced for debugging

## Deployment Checklist

- [ ] Review device compatibility matrix above
- [ ] Test on all target device types before production
- [ ] Monitor console logs during initial deployment
- [ ] Verify no AC-3 only streams in content library (or accept audio loss on Nest Hub)
- [ ] Update Cast console receiver configuration
- [ ] Notify users of Nest Hub changes (AC-3 audio may be unavailable)
- [ ] Keep v1.1.3 as fallback during transition period

## License

Not specified. Check with repository owner for licensing terms.

## Support

For issues, questions, or contributions, please open an issue in the repository.

For security concerns or critical issues with specific devices, please provide:
- Device model and Chrome version
- Stream manifest (MPD/M3U8) URL or example
- Console log output with [DEVICE], [CODEC-VALIDATION], [AUDIO] messages
- Exact error behavior observed

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
**Last Updated:** 2026-09-04  
**Status:** Production-Ready (v1.2.0-SAFE)
