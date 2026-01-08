# Module Interface Contracts: Recording Components

**Feature**: 001-screen-recording
**Date**: 2026-01-02
**Purpose**: Define module interfaces and contracts between components

---

## Overview

This document defines the interfaces (traits) that each recording component must implement. These contracts ensure loose coupling and enable platform-specific implementations.

---

## 1. Screen Capturer Interface

**Purpose**: Defines contract for screen capture implementations

**Trait Definition**:

```rust
/// Platform-agnostic screen capture trait
pub trait ScreenCapturer: Send + Sync {
    /// Captures a single frame from the screen source
    ///
    /// # Returns
    /// Frame data as RGBA bytes with dimensions
    ///
    /// # Errors
    /// Returns error if capture fails, device unavailable, or permissions denied
    fn capture_frame(&mut self) -> Result<CapturedFrame, ScreenCaptureError>;

    /// Gets the dimensions of the capture area
    fn dimensions(&self) -> (u32, u32);

    /// Checks if the screen source is still valid (window not closed, display still connected)
    fn is_valid(&self) -> bool;

    /// Gets current frames per second (for monitoring performance)
    fn current_fps(&self) -> f32;
}

/// Represents a captured frame
pub struct CapturedFrame {
    pub data: Vec<u8>,      // RGBA bytes
    pub width: u32,
    pub height: u32,
    pub timestamp: Instant,  // Capture timestamp for A/V sync
}

/// Screen capture errors
#[derive(Error, Debug)]
pub enum ScreenCaptureError {
    #[error("Permission denied for screen capture. {platform_instructions}")]
    PermissionDenied { platform_instructions: String },

    #[error("Screen source no longer available: {0}")]
    SourceUnavailable(String),

    #[error("Capture failed: {0}")]
    CaptureFailed(String),

    #[error("Invalid screen source")]
    InvalidSource,
}
```

**Platform Implementations**:
- `MacOSScreenCapturer` (using xcap with CGDisplay backend)
- `LinuxScreenCapturer` (using xcap with X11/Wayland backend)
- `WindowsScreenCapturer` (using xcap with DXGI backend)

**Contract Guarantees**:
- ✅ MUST capture at least 30 fps when `capture_frame()` called in loop
- ✅ MUST return error with actionable message if permissions denied
- ✅ MUST detect when screen source becomes invalid (window closed)
- ✅ MUST provide timestamp for each frame (for A/V sync)
- ✅ MUST be thread-safe (Send + Sync)

---

## 2. Camera Capturer Interface

**Purpose**: Defines contract for camera/webcam capture

**Trait Definition**:

```rust
/// Platform-agnostic camera capture trait
pub trait CameraCapturer: Send + Sync {
    /// Captures a single frame from the camera
    ///
    /// # Returns
    /// Frame data as RGB bytes with dimensions
    ///
    /// # Errors
    /// Returns error if capture fails or device unavailable
    fn capture_frame(&mut self) -> Result<CameraFrame, CameraError>;

    /// Gets the camera resolution
    fn resolution(&self) -> (u32, u32);

    /// Checks if camera is still connected and accessible
    fn is_available(&self) -> bool;

    /// Lists all available camera devices
    fn list_devices() -> Result<Vec<CameraDevice>, CameraError>;
}

/// Represents a camera frame
pub struct CameraFrame {
    pub data: Vec<u8>,      // RGB bytes
    pub width: u32,
    pub height: u32,
    pub timestamp: Instant,  // Capture timestamp
}

/// Camera device info
pub struct CameraDevice {
    pub id: String,          // Platform-specific device ID
    pub name: String,        // Human-readable name
    pub resolutions: Vec<(u32, u32)>, // Supported resolutions
}

/// Camera errors
#[derive(Error, Debug)]
pub enum CameraError {
    #[error("Permission denied for camera access. {platform_instructions}")]
    PermissionDenied { platform_instructions: String },

    #[error("Camera device not found: {0}")]
    DeviceNotFound(String),

    #[error("Camera disconnected during capture")]
    Disconnected,

    #[error("Camera capture failed: {0}")]
    CaptureFailed(String),

    #[error("Unsupported resolution: {width}x{height}")]
    UnsupportedResolution { width: u32, height: u32 },
}
```

**Implementation**: `NokhwaCamera` (using nokhwa crate)

**Contract Guarantees**:
- ✅ MUST support 720p and 1080p if camera supports them
- ✅ MUST return error with actionable message if permissions denied
- ✅ MUST detect camera disconnection during capture
- ✅ MUST enumerate devices across all platforms
- ✅ MUST be thread-safe (Send + Sync)

---

## 3. Audio Capturer Interface

**Purpose**: Defines contract for microphone audio capture

**Trait Definition**:

```rust
/// Platform-agnostic audio capture trait
pub trait AudioCapturer: Send + Sync {
    /// Starts audio capture stream
    ///
    /// # Errors
    /// Returns error if device unavailable or permissions denied
    fn start(&mut self) -> Result<(), AudioError>;

    /// Stops audio capture stream
    fn stop(&mut self) -> Result<(), AudioError>;

    /// Drains captured audio samples from buffer
    ///
    /// # Returns
    /// Vec of f32 samples (normalized -1.0 to 1.0)
    fn drain_samples(&mut self) -> Vec<f32>;

    /// Gets current audio level (RMS) for UI indicator
    ///
    /// # Returns
    /// Level from 0.0 (silence) to 1.0 (maximum)
    fn current_level(&self) -> f32;

    /// Gets audio configuration
    fn config(&self) -> AudioConfig;

    /// Checks if audio device is still available
    fn is_available(&self) -> bool;

    /// Lists all available audio input devices
    fn list_devices() -> Result<Vec<AudioDevice>, AudioError>;
}

/// Audio configuration
#[derive(Clone, Debug)]
pub struct AudioConfig {
    pub sample_rate: u32,    // e.g., 44100, 48000
    pub channels: u16,       // 1 = mono, 2 = stereo
}

/// Audio device info
pub struct AudioDevice {
    pub id: String,          // Platform-specific device ID
    pub name: String,        // Human-readable name
    pub supported_configs: Vec<AudioConfig>,
}

/// Audio errors
#[derive(Error, Debug)]
pub enum AudioError {
    #[error("Permission denied for microphone access. {platform_instructions}")]
    PermissionDenied { platform_instructions: String },

    #[error("Audio device not found: {0}")]
    DeviceNotFound(String),

    #[error("Audio device disconnected")]
    Disconnected,

    #[error("Audio stream failed: {0}")]
    StreamFailed(String),

    #[error("Unsupported audio configuration")]
    UnsupportedConfig,
}
```

**Implementation**: `CpalAudio` (using cpal crate)

**Contract Guarantees**:
- ✅ MUST support 44100 Hz and 48000 Hz sample rates
- ✅ MUST support mono (1 channel) audio
- ✅ MUST provide real-time level metering (current_level)
- ✅ MUST buffer audio samples for encoding
- ✅ MUST return error with actionable message if permissions denied
- ✅ MUST detect device disconnection
- ✅ MUST be thread-safe (Send + Sync)

---

## 4. Video Encoder Interface

**Purpose**: Defines contract for video encoding and muxing

**Trait Definition**:

```rust
/// Video encoder trait for encoding and muxing
pub trait VideoEncoder: Send {
    /// Adds a video frame to the encoding queue
    ///
    /// # Arguments
    /// * `frame` - Screen frame (RGBA bytes)
    /// * `overlay` - Optional camera overlay frame (RGB bytes)
    ///
    /// # Errors
    /// Returns error if encoding fails
    fn encode_frame(
        &mut self,
        frame: &CapturedFrame,
        overlay: Option<&CameraFrame>,
    ) -> Result<(), EncodingError>;

    /// Adds audio samples to the encoding queue
    ///
    /// # Arguments
    /// * `samples` - Audio samples (f32, normalized -1.0 to 1.0)
    ///
    /// # Errors
    /// Returns error if encoding fails
    fn encode_audio(&mut self, samples: &[f32]) -> Result<(), EncodingError>;

    /// Finalizes encoding and writes output file
    ///
    /// # Arguments
    /// * `output_path` - Where to save the video file
    ///
    /// # Returns
    /// Recording output metadata
    ///
    /// # Errors
    /// Returns error if finalization or file write fails
    fn finalize(self, output_path: &Path) -> Result<RecordingOutput, EncodingError>;

    /// Gets encoding progress (0.0 to 1.0)
    fn progress(&self) -> f32;

    /// Checks if encoder has sufficient disk space
    fn check_disk_space(&self, output_path: &Path) -> Result<(), EncodingError>;
}

/// Encoding errors
#[derive(Error, Debug)]
pub enum EncodingError {
    #[error("Video encoding failed: {0}")]
    VideoEncodingFailed(String),

    #[error("Audio encoding failed: {0}")]
    AudioEncodingFailed(String),

    #[error("Insufficient disk space: {needed} bytes required, {available} available")]
    InsufficientDiskSpace { needed: u64, available: u64 },

    #[error("File write failed: {0}")]
    FileWriteFailed(#[from] std::io::Error),

    #[error("Unsupported codec or format combination")]
    UnsupportedFormat,

    #[error("A/V synchronization error: {0}")]
    SyncError(String),
}
```

**Implementation**: `FfmpegEncoder` (using ffmpeg-next crate)

**Contract Guarantees**:
- ✅ MUST composite camera overlay onto screen frame if provided
- ✅ MUST synchronize audio and video streams (< 100ms drift)
- ✅ MUST encode at ≥2x real-time speed
- ✅ MUST check disk space before encoding
- ✅ MUST produce playable MP4 files
- ✅ MUST support H.264 codec (minimum)
- ✅ Thread-safe for video frames, audio may require mutex

---

## 5. Recording Session Manager Interface

**Purpose**: Orchestrates all recording components

**Trait Definition**:

```rust
/// High-level recording session management
pub trait RecordingSessionManager: Send + Sync {
    /// Starts a new recording session
    ///
    /// # Arguments
    /// * `config` - Recording configuration
    ///
    /// # Errors
    /// Returns error if any component fails to initialize
    fn start(&mut self, config: RecordingConfig) -> Result<(), SessionError>;

    /// Pauses the current recording
    fn pause(&mut self) -> Result<(), SessionError>;

    /// Resumes a paused recording
    fn resume(&mut self) -> Result<(), SessionError>;

    /// Stops the recording and prepares for save
    fn stop(&mut self) -> Result<(), SessionError>;

    /// Saves the recording to specified path
    ///
    /// # Returns
    /// Recording output metadata
    fn save(&mut self, output_path: &Path) -> Result<RecordingOutput, SessionError>;

    /// Cancels the recording without saving
    fn cancel(&mut self) -> Result<(), SessionError>;

    /// Gets current recording state
    fn state(&self) -> RecordingState;

    /// Gets elapsed recording time (excluding pauses)
    fn elapsed_time(&self) -> Duration;

    /// Gets current audio level for UI indicator
    fn audio_level(&self) -> f32;

    /// Checks if session is healthy (all devices available)
    fn is_healthy(&self) -> bool;
}

/// Recording configuration
pub struct RecordingConfig {
    pub screen_source: ScreenSource,
    pub camera_config: Option<CameraConfig>,
    pub audio_config: Option<AudioConfig>,
    pub output_format: VideoFormat,
    pub quality: QualityPreset,
}

pub struct CameraConfig {
    pub device_id: String,
    pub position: Position,
    pub size: Size,
    pub shape: OverlayShape,
}

/// Session errors
#[derive(Error, Debug)]
pub enum SessionError {
    #[error("Screen capture error: {0}")]
    ScreenCapture(#[from] ScreenCaptureError),

    #[error("Camera error: {0}")]
    Camera(#[from] CameraError),

    #[error("Audio error: {0}")]
    Audio(#[from] AudioError),

    #[error("Encoding error: {0}")]
    Encoding(#[from] EncodingError),

    #[error("Invalid state transition: cannot {action} while {current_state}")]
    InvalidStateTransition { action: String, current_state: String },

    #[error("Session not started")]
    NotStarted,

    #[error("Session already started")]
    AlreadyStarted,
}
```

**Implementation**: `RecordingSession` in `src/recording/session.rs`

**Contract Guarantees**:
- ✅ MUST enforce valid state transitions (per data model)
- ✅ MUST coordinate all components (screen, camera, audio, encoder)
- ✅ MUST handle component failures gracefully
- ✅ MUST track pause segments for accurate elapsed time
- ✅ MUST be thread-safe (Send + Sync)

---

## 6. Configuration Storage Interface

**Purpose**: Persists user settings between sessions

**Trait Definition**:

```rust
/// Configuration persistence trait
pub trait ConfigStorage: Send + Sync {
    /// Loads settings from disk
    ///
    /// # Returns
    /// Loaded settings or default settings if file doesn't exist
    ///
    /// # Errors
    /// Returns error if file is corrupted or unreadable
    fn load() -> Result<ApplicationSettings, ConfigError>;

    /// Saves settings to disk
    ///
    /// # Errors
    /// Returns error if write fails
    fn save(&self, settings: &ApplicationSettings) -> Result<(), ConfigError>;

    /// Resets to default settings
    fn reset_to_defaults(&self) -> ApplicationSettings;

    /// Gets platform-specific config directory
    fn config_dir() -> PathBuf;
}

/// Configuration errors
#[derive(Error, Debug)]
pub enum ConfigError {
    #[error("Failed to read config file: {0}")]
    ReadFailed(#[from] std::io::Error),

    #[error("Failed to parse config file: {0}")]
    ParseFailed(String),

    #[error("Failed to write config file: {0}")]
    WriteFailed(String),

    #[error("Config directory not accessible")]
    DirectoryNotAccessible,
}
```

**Implementation**: `TomlConfigStorage` in `src/config/storage.rs`

**Contract Guarantees**:
- ✅ MUST use platform-appropriate config directories
- ✅ MUST create default config if missing
- ✅ MUST validate loaded config values
- ✅ MUST handle corrupted files gracefully (fall back to defaults)
- ✅ MUST be thread-safe (Send + Sync)

---

## Module Dependencies

```
RecordingSessionManager
    ├── ScreenCapturer (xcap)
    ├── CameraCapturer (nokhwa) [optional]
    ├── AudioCapturer (cpal) [optional]
    └── VideoEncoder (ffmpeg-next)

ConfigStorage (standalone)
```

---

## Cross-Cutting Concerns

### Error Handling

All errors MUST:
- Use `thiserror` for derive macros
- Provide actionable user-facing messages
- Include platform-specific instructions for permissions
- Support error chaining via `#[from]`

### Platform-Specific Instructions

Each `PermissionDenied` error MUST include:

**macOS**:
```rust
"Enable in System Preferences → Security & Privacy → Privacy → [Screen Recording/Camera/Microphone]"
```

**Linux**:
```rust
"Grant permissions via your desktop environment settings or check PipeWire/PulseAudio access"
```

**Windows**:
```rust
"Enable in Settings → Privacy → [Screen/Camera/Microphone] and allow desktop apps"
```

### Thread Safety

All traits require `Send + Sync` except:
- `VideoEncoder`: Only `Send` (may have non-thread-safe FFmpeg internals)

Use `Arc<Mutex<>>` for shared mutable state only when necessary (e.g., audio buffer).

### Timestamps

All captured frames (screen, camera) and audio samples MUST include `Instant` timestamps for accurate A/V synchronization.

---

## Testing Contracts

Each implementation MUST provide:

1. **Unit Tests** (`#[cfg(test)]` modules):
   - Test each trait method with valid inputs
   - Test error conditions (permissions denied, device disconnected)
   - Test state transitions (for SessionManager)

2. **Integration Tests** (`tests/` directory):
   - Test complete recording workflows
   - Test component interactions
   - Test cross-platform compatibility

3. **Benchmarks** (`benches/` directory):
   - Verify performance guarantees (fps, encoding speed, CPU overhead)
   - Measure A/V sync accuracy

---

## Summary

These contracts ensure:
- ✅ **Loose coupling**: Components interact via traits
- ✅ **Platform abstraction**: Platform-specific implementations behind unified interfaces
- ✅ **Testability**: Traits enable mocking for unit tests
- ✅ **Safety**: All interfaces enforce thread safety where needed
- ✅ **Error handling**: Consistent error types with actionable messages
- ✅ **Performance**: Contracts specify performance guarantees

All implementations MUST satisfy the guarantees listed in their respective contract sections.
