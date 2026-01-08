# Data Model: Screen Recording with Camera Overlay

**Feature**: 001-screen-recording
**Date**: 2026-01-02
**Purpose**: Define data structures and their relationships for the screen recording application

---

## Overview

This application has **no persistent database**. All data is either:
1. **Transient** - exists only during recording session (in-memory state)
2. **File-based** - video files and configuration files

The data model focuses on in-memory structures used during runtime.

---

## 1. Recording Session

**Purpose**: Represents an active or completed recording session

**State Diagram**:
```
Idle → Recording → Paused → Recording → Stopped → Saved
  ↓                           ↓
  └──────────────────────────→ Cancelled
```

**Fields**:

| Field | Type | Description | Validation |
|-------|------|-------------|------------|
| `id` | `Uuid` | Unique session identifier | Auto-generated |
| `state` | `RecordingState` | Current state (Idle/Recording/Paused/Stopped) | Enum validation |
| `start_time` | `Option<Instant>` | When recording started | None if not started |
| `elapsed_time` | `Duration` | Total recording duration (excluding pauses) | >= 0 |
| `pause_segments` | `Vec<(Instant, Instant)>` | List of pause start/end times | Must not overlap |
| `screen_source` | `ScreenSource` | What is being recorded | Required |
| `camera_enabled` | `bool` | Whether camera overlay is active | Default: false |
| `microphone_enabled` | `bool` | Whether microphone is recording | Default: false |
| `output_path` | `Option<PathBuf>` | Where to save the recording | None until user selects |

**Methods**:
- `start()` → Transitions from Idle to Recording
- `pause()` → Transitions from Recording to Paused
- `resume()` → Transitions from Paused to Recording
- `stop()` → Transitions to Stopped
- `get_elapsed()` → Returns Duration excluding pauses
- `is_active()` → Returns true if Recording or Paused

**Relationships**:
- Has one `ScreenSource` (what to capture)
- Has zero or one `CameraOverlay` (if camera enabled)
- Has zero or one `AudioInput` (if microphone enabled)
- Produces one `RecordingOutput` (video file)

---

## 2. Recording State

**Purpose**: Enum representing the current state of a recording session

**Values**:

```rust
enum RecordingState {
    Idle,       // No recording in progress, ready to start
    Recording,  // Actively recording
    Paused,     // Recording paused, can be resumed
    Stopped,    // Recording stopped, ready to save
    Saving,     // Encoding and saving to file
    Completed,  // Successfully saved
    Cancelled,  // User cancelled before saving
    Error(String), // Error occurred, with message
}
```

**State Transitions**:
- `Idle → Recording`: User clicks "Start Recording"
- `Recording → Paused`: User clicks "Pause"
- `Paused → Recording`: User clicks "Resume"
- `Recording → Stopped`: User clicks "Stop"
- `Stopped → Saving`: User selects save location
- `Saving → Completed`: File successfully saved
- `Any → Cancelled`: User cancels
- `Any → Error`: Error occurs

**Validation**:
- Cannot transition from `Completed` or `Cancelled` (terminal states)
- Cannot transition from `Idle` to `Paused`
- Cannot transition from `Stopped` back to `Recording`

---

## 3. Screen Source

**Purpose**: Defines what screen content to capture

**Variants**:

```rust
enum ScreenSource {
    EntireScreen {
        display_id: DisplayId,
    },
    SpecificWindow {
        window_id: WindowId,
        window_title: String,
    },
}
```

**Fields**:

| Field | Type | Description | Validation |
|-------|------|-------------|------------|
| `display_id` | `u32` or platform-specific | Identifies which display/monitor | Must exist on system |
| `window_id` | `u64` or platform-specific | Platform-specific window identifier | Must be valid window |
| `window_title` | `String` | Human-readable window name | For display purposes |

**Methods**:
- `list_displays()` → Static method to enumerate available displays
- `list_windows()` → Static method to enumerate capturable windows
- `is_valid()` → Checks if source still exists

**Validation Rules**:
- Display ID must correspond to connected display
- Window ID must correspond to existing window
- If window is closed during recording, handle gracefully (FR-014 edge case)

---

## 4. Camera Overlay

**Purpose**: Configuration for camera feed overlay display

**Fields**:

| Field | Type | Description | Validation | Default |
|-------|------|-------------|------------|---------|
| `device_id` | `String` | Camera device identifier | Must be available device | Default device |
| `device_name` | `String` | Human-readable camera name | For UI display | - |
| `enabled` | `bool` | Whether overlay is visible | - | false |
| `position` | `Position` | Overlay position on screen | Within screen bounds | Bottom-right |
| `size` | `Size` | Overlay dimensions | Min 100x100, max 800x600 | 320x240 |
| `shape` | `OverlayShape` | Display shape (circle/rectangle) | - | Circle |
| `resolution` | `(u32, u32)` | Camera capture resolution | Supported by device | 1280x720 |

**Position Type**:

```rust
struct Position {
    x: f32,  // 0.0 = left edge, 1.0 = right edge (normalized)
    y: f32,  // 0.0 = top edge, 1.0 = bottom edge (normalized)
}
```

**Size Type**:

```rust
struct Size {
    width: u32,   // In pixels
    height: u32,  // In pixels
}
```

**Overlay Shape**:

```rust
enum OverlayShape {
    Circle,         // Circular camera feed (most common)
    Rectangle,      // Rectangular camera feed
    RoundedRect(u32), // Rounded rectangle with corner radius
}
```

**Methods**:
- `move_to(x, y)` → Updates position
- `resize(width, height)` → Updates size with validation
- `set_device(device_id)` → Changes camera device
- `toggle_enabled()` → Enables/disables overlay

**Validation Rules**:
- Position coordinates must be 0.0 to 1.0 (normalized)
- Size must be within min/max bounds
- Device ID must be available camera
- Resolution must be supported by selected camera

---

## 5. Audio Input

**Purpose**: Configuration for microphone audio capture

**Fields**:

| Field | Type | Description | Validation | Default |
|-------|------|-------------|------------|---------|
| `device_id` | `String` | Microphone device identifier | Must be available device | Default device |
| `device_name` | `String` | Human-readable device name | For UI display | - |
| `enabled` | `bool` | Whether microphone is recording | - | false |
| `sample_rate` | `u32` | Sample rate in Hz | 44100 or 48000 | 48000 |
| `channels` | `u16` | Number of channels | 1 (mono) or 2 (stereo) | 1 (mono) |
| `current_level` | `f32` | Real-time audio level (RMS) | 0.0 to 1.0 | 0.0 |
| `buffer` | `Arc<Mutex<Vec<f32>>>` | Captured audio samples | - | Empty |

**Methods**:
- `start_capture()` → Begins capturing audio
- `stop_capture()` → Stops capturing audio
- `get_level()` → Returns current audio level for UI indicator
- `set_device(device_id)` → Changes microphone device
- `drain_buffer()` → Retrieves and clears captured audio samples

**Validation Rules**:
- Device ID must be valid and available
- Sample rate must be 44100 or 48000 Hz
- Channels must be 1 or 2
- Current level must be 0.0 to 1.0 (RMS normalized)

---

## 6. Recording Output

**Purpose**: Represents the saved video file

**Fields**:

| Field | Type | Description | Validation |
|-------|------|-------------|------------|
| `file_path` | `PathBuf` | Absolute path to saved file | Must be writable location |
| `format` | `VideoFormat` | Output format (MP4, etc.) | Must be supported |
| `codec` | `VideoCodec` | Video codec used | H.264, H.265, VP9 |
| `audio_codec` | `Option<AudioCodec>` | Audio codec if audio included | AAC, Opus, etc. |
| `resolution` | `(u32, u32)` | Video resolution (width, height) | Matches screen source |
| `frame_rate` | `u32` | Frames per second | Typically 30 or 60 |
| `duration` | `Duration` | Total video duration | Matches recording duration |
| `file_size` | `u64` | File size in bytes | > 0 |
| `created_at` | `SystemTime` | When file was created | - |

**Video Format**:

```rust
enum VideoFormat {
    MP4,
    MKV,
    WebM,
}
```

**Video Codec**:

```rust
enum VideoCodec {
    H264,      // Baseline/main/high profiles
    H265,      // HEVC
    VP9,       // WebM codec
}
```

**Audio Codec**:

```rust
enum AudioCodec {
    AAC,       // Most common for MP4
    Opus,      // Modern, efficient
    MP3,       // Legacy support
}
```

**Methods**:
- `exists()` → Checks if file still exists
- `delete()` → Permanently deletes the file
- `get_metadata()` → Retrieves file system metadata
- `verify_playback()` → Validates file is playable

**Validation Rules**:
- File path must be absolute and in writable location
- Parent directory must exist
- Sufficient disk space required (estimate: duration * bitrate)
- File extension must match format (e.g., .mp4 for MP4)

---

## 7. Application Settings

**Purpose**: User preferences persisted between sessions

**Storage**: TOML config file at platform-specific location:
- **macOS**: `~/Library/Application Support/soufle/config.toml`
- **Linux**: `~/.config/soufle/config.toml` (XDG_CONFIG_HOME)
- **Windows**: `%APPDATA%\soufle\config.toml`

**Fields**:

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| `default_save_directory` | `PathBuf` | Where recordings are saved by default | User's Videos folder |
| `default_format` | `VideoFormat` | Default output format | MP4 |
| `default_codec` | `VideoCodec` | Default video codec | H264 |
| `default_quality` | `QualityPreset` | Encoding quality preset | Medium |
| `camera_device_id` | `Option<String>` | Last used camera | None (use system default) |
| `microphone_device_id` | `Option<String>` | Last used microphone | None (use system default) |
| `camera_position` | `Position` | Last camera overlay position | Bottom-right (0.8, 0.8) |
| `camera_size` | `Size` | Last camera overlay size | 320x240 |
| `camera_shape` | `OverlayShape` | Camera overlay shape | Circle |
| `keyboard_shortcuts` | `HashMap<Action, KeyBinding>` | Customizable shortcuts | Platform defaults |
| `show_timer` | `bool` | Show recording timer | true |
| `show_audio_meter` | `bool` | Show microphone level indicator | true |

**Quality Preset**:

```rust
enum QualityPreset {
    Low,      // Smaller files, lower quality (faster encoding)
    Medium,   // Balanced quality and file size
    High,     // Higher quality, larger files
    Lossless, // Maximum quality, very large files
}
```

**Action** (for keyboard shortcuts):

```rust
enum Action {
    StartRecording,
    StopRecording,
    PauseRecording,
    ResumeRecording,
    ToggleCamera,
    ToggleMicrophone,
}
```

**Methods**:
- `load()` → Static method to load from file (creates default if missing)
- `save()` → Writes current settings to file
- `reset_to_defaults()` → Restores default settings

**Validation Rules**:
- Default save directory must exist and be writable
- Device IDs must be valid (or None for system default)
- Camera position must be 0.0 to 1.0 normalized
- Camera size must be within min/max bounds

---

## 8. Error Types

**Purpose**: Typed errors for different failure scenarios (per constitution requirement)

**Error Hierarchy** (using `thiserror`):

```rust
#[derive(Error, Debug)]
pub enum SoufleError {
    #[error("Screen capture error: {0}")]
    ScreenCapture(String),

    #[error("Camera error: {0}")]
    Camera(String),

    #[error("Audio error: {0}")]
    Audio(String),

    #[error("Encoding error: {0}")]
    Encoding(String),

    #[error("File I/O error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Permission denied: {0}")]
    PermissionDenied(String),

    #[error("Device not found: {0}")]
    DeviceNotFound(String),

    #[error("Insufficient disk space: {needed} bytes required, {available} available")]
    InsufficientDiskSpace { needed: u64, available: u64 },

    #[error("Recording session error: {0}")]
    RecordingSession(String),

    #[error("Configuration error: {0}")]
    Config(String),
}
```

**Actionable Error Messages** (per constitution - FR-014):

Each error variant includes context for user-actionable messages:
- **PermissionDenied**: Include OS-specific instructions (e.g., "Enable in System Preferences → Privacy → Screen Recording")
- **DeviceNotFound**: List available devices or suggest checking connections
- **InsufficientDiskSpace**: Show required vs available space, suggest cleanup
- **ScreenCapture/Camera/Audio**: Provide specific failure reason and recovery steps

---

## Data Flow Diagram

```
┌─────────────────┐
│ Application     │
│ Settings        │
│ (TOML config)   │
└────────┬────────┘
         │ loads on startup
         ↓
┌─────────────────────────────────────────────┐
│         Recording Session (In-Memory)       │
│  ┌──────────────────────────────────────┐  │
│  │ State: Idle → Recording → Stopped    │  │
│  └──────────────────────────────────────┘  │
│         ↓             ↓            ↓        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │  Screen  │  │  Camera  │  │  Audio   │ │
│  │  Source  │  │  Overlay │  │  Input   │ │
│  └──────────┘  └──────────┘  └──────────┘ │
└────────────────────┬────────────────────────┘
                     │ encoding
                     ↓
             ┌───────────────┐
             │   Recording   │
             │    Output     │
             │ (MP4 file)    │
             └───────────────┘
```

---

## Validation Summary

| Entity | Key Validation Rules |
|--------|---------------------|
| Recording Session | State transitions must follow diagram, output path required before saving |
| Screen Source | Display/window must exist at capture time |
| Camera Overlay | Position 0.0-1.0, size within bounds, device must be available |
| Audio Input | Sample rate 44100/48000, channels 1-2, device must be available |
| Recording Output | Writable path, sufficient disk space, valid format/codec combination |
| Application Settings | Save directory writable, device IDs valid or None |
| Error Types | All errors must have actionable messages for users |

---

## Platform-Specific Considerations

### macOS
- **Display IDs**: Use CGDirectDisplayID (u32)
- **Window IDs**: Use CGWindowID (u32)
- **Camera/Microphone**: Require Info.plist privacy permissions
- **Config path**: `~/Library/Application Support/soufle/`

### Linux
- **Display IDs**: X11 screen number or Wayland output ID
- **Window IDs**: X11 Window ID (u64) or Wayland surface ID
- **Audio**: ALSA or JACK device names (strings)
- **Config path**: `$XDG_CONFIG_HOME/soufle/` or `~/.config/soufle/`

### Windows
- **Display IDs**: Monitor handle (HMONITOR)
- **Window IDs**: Window handle (HWND)
- **Audio**: WASAPI device endpoint ID
- **Config path**: `%APPDATA%\soufle\`

Use conditional compilation (`#[cfg(target_os = "...")]`) for platform-specific types while maintaining unified traits.

---

## Notes

- All in-memory data structures use Rust ownership for safety
- No shared mutable state except where required (e.g., audio buffer uses `Arc<Mutex<>>`)
- All file paths use `PathBuf` for cross-platform compatibility
- All time measurements use `std::time::Instant` (monotonic) for accuracy
- User-facing times use `std::time::SystemTime` for wall-clock display
