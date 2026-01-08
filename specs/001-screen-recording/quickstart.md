# Quickstart: Screen Recording with Camera Overlay

**Feature**: 001-screen-recording
**Date**: 2026-01-02
**Purpose**: Step-by-step implementation guide for developers

---

## Prerequisites

- Rust 1.75+ (stable channel)
- Platform-specific build tools:
  - **macOS**: Xcode Command Line Tools
  - **Linux**: `build-essential`, `libx11-dev`, `libxrandr-dev`, `libasound2-dev`, `pkg-config`
  - **Windows**: MSVC toolchain or MinGW

---

## Initial Project Setup

### 1. Initialize Cargo Project

```bash
# Already initialized - skip this step if repo exists
cargo new soufle
cd soufle
```

### 2. Update Cargo.toml

```toml
[package]
name = "soufle"
version = "0.1.0"
edition = "2021"
rust-version = "1.75"

[dependencies]
# GUI framework
iced = { version = "0.12", features = ["tokio", "image"] }

# Screen capture - cross-platform native backends
xcap = "0.0.9"

# Video encoding and muxing
ffmpeg-next = "7.0"

# Camera capture - cross-platform
nokhwa = { version = "0.10", features = ["input-native"] }

# Audio capture - cross-platform
cpal = "0.15"

# Configuration serialization
serde = { version = "1.0", features = ["derive"] }
toml = "0.8"

# Platform-specific paths
dirs = "5.0"

# Error handling (per constitution)
thiserror = "1.0"

# Logging (per constitution)
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }

# Utilities
uuid = { version = "1.0", features = ["v4"] }

[dev-dependencies]
# Testing
criterion = "0.5"  # Benchmarks

[build-dependencies]
# If needed for FFmpeg linking
pkg-config = "0.3"

[[bench]]
name = "recording"
harness = false

[profile.release]
# Per constitution: maximum optimization
lto = "fat"
codegen-units = 1
opt-level = 3
```

### 3. Create Project Structure

```bash
# Create source directories
mkdir -p src/ui/screens
mkdir -p src/ui/widgets
mkdir -p src/recording
mkdir -p src/platform
mkdir -p src/config

# Create test directories
mkdir -p tests/integration
mkdir -p tests/unit

# Create bench directory
mkdir -p benches

# Create config files
touch rustfmt.toml
touch .clippy.toml
```

---

## Configuration Files

### rustfmt.toml

```toml
edition = "2021"
max_width = 100
use_small_heuristics = "Default"
```

### .clippy.toml

```toml
# No warnings allowed (per constitution)
warn-on-all-wildcard-imports = true
```

---

## Implementation Order (Phased Approach)

### Phase 1: MVP - Basic Screen Recording (P1)

**Goal**: Record screen and save to file (minimal viable product)

#### Step 1.1: Set up logging and error types

**File**: `src/error.rs`

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum SoufleError {
    #[error("Screen capture error: {0}")]
    ScreenCapture(String),

    #[error("Encoding error: {0}")]
    Encoding(String),

    #[error("File I/O error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Permission denied: {0}. {platform_instructions}")]
    PermissionDenied {
        reason: String,
        platform_instructions: String,
    },

    #[error("Insufficient disk space: {needed} bytes required, {available} available")]
    InsufficientDiskSpace { needed: u64, available: u64 },
}

pub type Result<T> = std::result::Result<T, SoufleError>;
```

**File**: `src/main.rs`

```rust
mod error;

use tracing_subscriber;

fn main() {
    // Initialize logging (per constitution - use tracing, not println!)
    tracing_subscriber::fmt::init();

    tracing::info!("Starting Soufle v{}", env!("CARGO_PKG_VERSION"));

    // TODO: Launch iced application
}
```

**Test**: `cargo run` - should print log message

---

#### Step 1.2: Implement screen capture trait and xcap integration

**File**: `src/recording/screen.rs`

```rust
use crate::error::{Result, SoufleError};
use std::time::Instant;

/// Platform-agnostic screen capture trait
pub trait ScreenCapturer: Send + Sync {
    fn capture_frame(&mut self) -> Result<CapturedFrame>;
    fn dimensions(&self) -> (u32, u32);
    fn is_valid(&self) -> bool;
    fn current_fps(&self) -> f32;
}

/// Captured screen frame
pub struct CapturedFrame {
    pub data: Vec<u8>,      // RGBA bytes
    pub width: u32,
    pub height: u32,
    pub timestamp: Instant,
}

/// xcap-based implementation
pub struct XCapScreenCapturer {
    // TODO: xcap types
}

impl ScreenCapturer for XCapScreenCapturer {
    fn capture_frame(&mut self) -> Result<CapturedFrame> {
        // TODO: Use xcap to capture frame
        todo!()
    }

    fn dimensions(&self) -> (u32, u32) {
        todo!()
    }

    fn is_valid(&self) -> bool {
        todo!()
    }

    fn current_fps(&self) -> f32 {
        todo!()
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_screen_capturer() {
        // TODO: Test screen capture
    }
}
```

**Test**: Write unit tests that verify screen capture works

---

#### Step 1.3: Implement video encoder trait and FFmpeg integration

**File**: `src/recording/encoder.rs`

```rust
use crate::error::Result;
use crate::recording::screen::CapturedFrame;
use std::path::Path;

pub trait VideoEncoder: Send {
    fn encode_frame(&mut self, frame: &CapturedFrame) -> Result<()>;
    fn finalize(self, output_path: &Path) -> Result<()>;
}

pub struct FfmpegEncoder {
    // TODO: ffmpeg-next types
}

impl VideoEncoder for FfmpegEncoder {
    fn encode_frame(&mut self, frame: &CapturedFrame) -> Result<()> {
        // TODO: Encode frame with FFmpeg
        todo!()
    }

    fn finalize(self, output_path: &Path) -> Result<()> {
        // TODO: Finalize and write MP4 file
        todo!()
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_encoding() {
        // TODO: Test encoding
    }
}
```

---

#### Step 1.4: Implement basic iced UI

**File**: `src/ui/app.rs`

```rust
use iced::{Application, Command, Element, Settings, Theme};

#[derive(Debug, Default)]
pub struct App {
    // TODO: Add state
}

#[derive(Debug, Clone)]
pub enum Message {
    StartRecording,
    StopRecording,
}

impl Application for App {
    type Executor = iced::executor::Default;
    type Message = Message;
    type Theme = Theme;
    type Flags = ();

    fn new(_flags: ()) -> (Self, Command<Message>) {
        (App::default(), Command::none())
    }

    fn title(&self) -> String {
        String::from("Soufle - Screen Recorder")
    }

    fn update(&mut self, message: Message) -> Command<Message> {
        match message {
            Message::StartRecording => {
                tracing::info!("Start recording clicked");
                // TODO: Start recording
                Command::none()
            }
            Message::StopRecording => {
                tracing::info!("Stop recording clicked");
                // TODO: Stop recording
                Command::none()
            }
        }
    }

    fn view(&self) -> Element<Message> {
        use iced::widget::{button, column, text};

        column![
            text("Soufle - Screen Recorder").size(30),
            button("Start Recording")
                .on_press(Message::StartRecording),
            button("Stop Recording")
                .on_press(Message::StopRecording),
        ]
        .padding(20)
        .into()
    }
}
```

**File**: `src/main.rs` (update)

```rust
mod ui;

fn main() {
    tracing_subscriber::fmt::init();

    ui::app::App::run(Settings::default())
        .expect("Failed to start application");
}
```

**Test**: `cargo run` - GUI window should open with buttons

---

#### Step 1.5: Integrate recording session

**File**: `src/recording/session.rs`

```rust
use crate::error::Result;
use crate::recording::{screen::ScreenCapturer, encoder::VideoEncoder};
use std::path::Path;
use std::time::Duration;

pub struct RecordingSession {
    screen_capturer: Box<dyn ScreenCapturer>,
    encoder: Box<dyn VideoEncoder>,
    // TODO: Add more state
}

impl RecordingSession {
    pub fn new(
        screen_capturer: Box<dyn ScreenCapturer>,
        encoder: Box<dyn VideoEncoder>,
    ) -> Self {
        Self {
            screen_capturer,
            encoder,
        }
    }

    pub fn start(&mut self) -> Result<()> {
        // TODO: Start capture loop
        todo!()
    }

    pub fn stop(&mut self) -> Result<()> {
        // TODO: Stop capture
        todo!()
    }

    pub fn save(&mut self, path: &Path) -> Result<()> {
        // TODO: Save recording
        todo!()
    }

    pub fn elapsed_time(&self) -> Duration {
        // TODO: Return elapsed time
        Duration::default()
    }
}
```

**Test**: Integration test recording full workflow

---

### Phase 2: Camera Overlay (P2)

**Goal**: Add camera overlay functionality

#### Step 2.1: Implement camera capture with nokhwa

**File**: `src/recording/camera.rs`

```rust
use crate::error::Result;
use std::time::Instant;

pub trait CameraCapturer: Send + Sync {
    fn capture_frame(&mut self) -> Result<CameraFrame>;
    fn resolution(&self) -> (u32, u32);
    fn is_available(&self) -> bool;
}

pub struct CameraFrame {
    pub data: Vec<u8>,  // RGB bytes
    pub width: u32,
    pub height: u32,
    pub timestamp: Instant,
}

pub struct NokhwaCamera {
    // TODO: nokhwa types
}

impl CameraCapturer for NokhwaCamera {
    fn capture_frame(&mut self) -> Result<CameraFrame> {
        // TODO: Capture from nokhwa
        todo!()
    }

    fn resolution(&self) -> (u32, u32) {
        todo!()
    }

    fn is_available(&self) -> bool {
        todo!()
    }
}
```

---

#### Step 2.2: Add camera overlay compositing to encoder

Update `encoder.rs` to composite camera overlay onto screen frames before encoding.

---

### Phase 3: Audio Recording (P2)

**Goal**: Add microphone audio capture

#### Step 3.1: Implement audio capture with cpal

**File**: `src/recording/audio.rs`

```rust
use crate::error::Result;

pub trait AudioCapturer: Send + Sync {
    fn start(&mut self) -> Result<()>;
    fn stop(&mut self) -> Result<()>;
    fn drain_samples(&mut self) -> Vec<f32>;
    fn current_level(&self) -> f32;
}

pub struct CpalAudio {
    // TODO: cpal types
}

impl AudioCapturer for CpalAudio {
    fn start(&mut self) -> Result<()> {
        // TODO: Start cpal stream
        todo!()
    }

    fn stop(&mut self) -> Result<()> {
        todo!()
    }

    fn drain_samples(&mut self) -> Vec<f32> {
        todo!()
    }

    fn current_level(&self) -> f32 {
        // TODO: Calculate RMS for level meter
        todo!()
    }
}
```

---

#### Step 3.2: Add audio encoding to FFmpeg encoder

Update encoder to accept audio samples and mux with video.

---

### Phase 4: Recording Controls (P3)

**Goal**: Add pause/resume and keyboard shortcuts

#### Step 4.1: Implement pause/resume state management

Update `RecordingSession` to track pause segments.

#### Step 4.2: Add keyboard shortcuts

Use iced subscriptions for global keyboard events.

---

## Development Workflow

### Run (Debug)

```bash
cargo run
```

### Run (Release)

```bash
cargo build --release
./target/release/soufle
```

### Tests

```bash
# Unit tests
cargo test

# Integration tests
cargo test --test integration

# With coverage (requires cargo-tarpaulin)
cargo install cargo-tarpaulin
cargo tarpaulin --out Html
```

### Linting

```bash
# Format code (per constitution)
cargo fmt

# Lint with zero warnings (per constitution)
cargo clippy -- -D warnings

# Check compilation
cargo check
```

### Benchmarks

```bash
cargo bench
```

---

## Platform-Specific Setup

### macOS

**Info.plist** (required for permissions):

Create `Info.plist` in project root:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSCameraUsageDescription</key>
    <string>Soufle needs camera access to record your camera overlay.</string>
    <key>NSMicrophoneUsageDescription</key>
    <string>Soufle needs microphone access to record audio narration.</string>
    <key>NSScreenCaptureUsageDescription</key>
    <string>Soufle needs screen recording permission to capture your screen.</string>
</dict>
</plist>
```

Include in bundle when distributing.

---

### Linux

**Dependencies**:

```bash
# Ubuntu/Debian
sudo apt install build-essential libx11-dev libxrandr-dev libasound2-dev pkg-config

# Fedora
sudo dnf install gcc libX11-devel libXrandr-devel alsa-lib-devel pkg-config
```

**FFmpeg**:

```bash
sudo apt install ffmpeg libavcodec-dev libavformat-dev libavutil-dev
```

---

### Windows

**FFmpeg**:

Download FFmpeg shared libraries from https://www.gyan.dev/ffmpeg/builds/ and place in system PATH or link statically.

---

## Testing Checklist (MVP)

Before merging Phase 1:

- [ ] Screen capture works on all platforms (macOS, Linux, Windows)
- [ ] Video encoding produces playable MP4 files
- [ ] GUI starts and responds to clicks
- [ ] Recording can be started and stopped
- [ ] Saved videos are playable in VLC/QuickTime/Windows Media Player
- [ ] All tests pass: `cargo test`
- [ ] Zero clippy warnings: `cargo clippy -- -D warnings`
- [ ] Code formatted: `cargo fmt --check`
- [ ] Performance benchmarks pass (30+ fps, <5% CPU)

---

## Troubleshooting

### FFmpeg linking errors

**Linux**:
```bash
export PKG_CONFIG_PATH=/usr/lib/pkgconfig
cargo clean && cargo build
```

**macOS**:
```bash
brew install ffmpeg
export PKG_CONFIG_PATH=/opt/homebrew/lib/pkgconfig
```

### Permission errors

**macOS**: Grant permissions in System Preferences → Security & Privacy → Privacy

**Linux**: Check PipeWire/PulseAudio permissions

**Windows**: Enable in Settings → Privacy

---

## Next Steps

After MVP is complete:

1. Add camera overlay (Phase 2)
2. Add audio capture (Phase 3)
3. Add recording controls (Phase 4)
4. Add settings persistence
5. Add platform-specific packaging (DMG, AppImage, MSI)
6. Set up CI/CD (GitHub Actions with matrix for all platforms)

---

## Reference Documentation

- **iced**: https://docs.rs/iced/
- **xcap**: https://docs.rs/xcap/
- **ffmpeg-next**: https://docs.rs/ffmpeg-next/
- **nokhwa**: https://docs.rs/nokhwa/
- **cpal**: https://docs.rs/cpal/

---

## Code Review Checklist (Per Constitution)

Before submitting PR:

- [ ] Zero compiler warnings
- [ ] `cargo clippy -- -D warnings` passes
- [ ] `cargo fmt` applied
- [ ] No `dbg!()`, `println!()`, or `eprintln!()` in production code
- [ ] All public APIs have rustdoc comments
- [ ] Tests pass on all platforms
- [ ] Benchmarks show no performance regression
- [ ] `cargo audit` passes (no vulnerabilities)
- [ ] New dependencies justified in Cargo.toml comments

---

This quickstart provides a clear path from empty project to working MVP. Follow the phases in order, completing tests and code review for each phase before proceeding to the next.
