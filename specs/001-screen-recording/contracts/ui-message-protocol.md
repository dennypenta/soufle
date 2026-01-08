# UI Message Protocol: Application State Management

**Feature**: 001-screen-recording
**Date**: 2026-01-02
**Purpose**: Define iced UI message types and state management contracts

---

## Overview

This document defines the message-passing protocol for the iced GUI application. Iced uses an Elm-style architecture where user interactions generate messages that update application state.

---

## Application State

**Purpose**: Root application state containing all UI and recording state

```rust
/// Main application state
pub struct App {
    // Recording state
    pub recording_session: Option<RecordingSession>,
    pub recording_state: RecordingState,
    pub elapsed_time: Duration,

    // Screen source selection
    pub available_displays: Vec<DisplayInfo>,
    pub available_windows: Vec<WindowInfo>,
    pub selected_source: Option<ScreenSource>,

    // Camera state
    pub camera_enabled: bool,
    pub available_cameras: Vec<CameraDevice>,
    pub selected_camera: Option<String>,
    pub camera_overlay: CameraOverlay,

    // Audio state
    pub microphone_enabled: bool,
    pub available_microphones: Vec<AudioDevice>,
    pub selected_microphone: Option<String>,
    pub audio_level: f32,

    // UI state
    pub current_screen: Screen,
    pub show_settings: bool,
    pub error_message: Option<String>,

    // Settings
    pub settings: ApplicationSettings,
}

/// Display info for screen selection
pub struct DisplayInfo {
    pub id: u32,
    pub name: String,
    pub resolution: (u32, u32),
    pub is_primary: bool,
}

/// Window info for window selection
pub struct WindowInfo {
    pub id: u64,
    pub title: String,
    pub owner_name: String, // Application name
}

/// UI screens
pub enum Screen {
    Home,        // Main recording controls
    Settings,    // Settings/preferences
    Preview,     // Recording preview (optional)
}
```

---

## Message Types

**Purpose**: All user interactions and system events as message types

```rust
/// Main application message enum
#[derive(Debug, Clone)]
pub enum Message {
    // Recording control messages
    StartRecording,
    StopRecording,
    PauseRecording,
    ResumeRecording,
    CancelRecording,

    // Screen source selection
    RefreshScreenSources,
    ScreenSourcesRefreshed {
        displays: Vec<DisplayInfo>,
        windows: Vec<WindowInfo>,
    },
    SelectDisplay(u32),
    SelectWindow(u64),

    // Camera control
    ToggleCamera(bool),
    RefreshCameras,
    CamerasRefreshed(Vec<CameraDevice>),
    SelectCamera(String),
    MoveCameraOverlay { x: f32, y: f32 },
    ResizeCameraOverlay { width: u32, height: u32 },
    ChangeCameraShape(OverlayShape),

    // Audio control
    ToggleMicrophone(bool),
    RefreshMicrophones,
    MicrophonesRefreshed(Vec<AudioDevice>),
    SelectMicrophone(String),
    AudioLevelUpdated(f32),

    // Recording session events
    RecordingStateChanged(RecordingState),
    ElapsedTimeUpdated(Duration),
    RecordingError(String),

    // File save
    SaveRecordingPrompt,
    SavePathSelected(Option<PathBuf>),
    RecordingSaved(RecordingOutput),

    // Settings
    OpenSettings,
    CloseSettings,
    UpdateSetting(SettingUpdate),
    SaveSettings,
    ResetSettingsToDefaults,

    // Navigation
    NavigateTo(Screen),

    // Error handling
    DismissError,

    // Periodic updates (subscriptions)
    Tick,  // Triggered every frame for time updates, audio levels
}

/// Setting update variants
#[derive(Debug, Clone)]
pub enum SettingUpdate {
    DefaultSaveDirectory(PathBuf),
    DefaultFormat(VideoFormat),
    DefaultCodec(VideoCodec),
    DefaultQuality(QualityPreset),
    CameraPosition(Position),
    CameraSize(Size),
    CameraShape(OverlayShape),
    KeyboardShortcut { action: Action, binding: KeyBinding },
    ShowTimer(bool),
    ShowAudioMeter(bool),
}

/// Keyboard shortcuts
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct KeyBinding {
    pub key: Key,
    pub modifiers: Modifiers,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct Modifiers {
    pub ctrl: bool,
    pub shift: bool,
    pub alt: bool,
    pub cmd: bool,  // macOS Command key
}

/// Key enum (simplified - actual implementation would use iced::keyboard::Key)
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum Key {
    Character(char),
    F1, F2, F3, /* ... F12 */
    Space,
    Enter,
    Escape,
}
```

---

## Message Handling (Update Function)

**Purpose**: Process messages and update application state

```rust
impl App {
    /// Update function - processes messages and returns commands
    pub fn update(&mut self, message: Message) -> Command<Message> {
        match message {
            // Recording controls
            Message::StartRecording => self.handle_start_recording(),
            Message::StopRecording => self.handle_stop_recording(),
            Message::PauseRecording => self.handle_pause_recording(),
            Message::ResumeRecording => self.handle_resume_recording(),
            Message::CancelRecording => self.handle_cancel_recording(),

            // Screen source selection
            Message::RefreshScreenSources => self.handle_refresh_screen_sources(),
            Message::ScreenSourcesRefreshed { displays, windows } => {
                self.available_displays = displays;
                self.available_windows = windows;
                Command::none()
            },
            Message::SelectDisplay(id) => {
                self.selected_source = Some(ScreenSource::EntireScreen { display_id: id });
                Command::none()
            },
            Message::SelectWindow(id) => {
                let window = self.available_windows.iter()
                    .find(|w| w.id == id)
                    .cloned();
                if let Some(window) = window {
                    self.selected_source = Some(ScreenSource::SpecificWindow {
                        window_id: id,
                        window_title: window.title,
                    });
                }
                Command::none()
            },

            // Camera controls
            Message::ToggleCamera(enabled) => {
                self.camera_enabled = enabled;
                if enabled && self.available_cameras.is_empty() {
                    // Lazy load cameras
                    return Command::perform(
                        async { CameraCapturer::list_devices() },
                        |result| match result {
                            Ok(cameras) => Message::CamerasRefreshed(cameras),
                            Err(e) => Message::RecordingError(e.to_string()),
                        }
                    );
                }
                Command::none()
            },

            Message::SelectCamera(device_id) => {
                self.selected_camera = Some(device_id.clone());
                self.camera_overlay.device_id = device_id;
                Command::none()
            },

            Message::MoveCameraOverlay { x, y } => {
                self.camera_overlay.position = Position { x, y };
                Command::none()
            },

            // Audio controls
            Message::ToggleMicrophone(enabled) => {
                self.microphone_enabled = enabled;
                if enabled && self.available_microphones.is_empty() {
                    return Command::perform(
                        async { AudioCapturer::list_devices() },
                        |result| match result {
                            Ok(mics) => Message::MicrophonesRefreshed(mics),
                            Err(e) => Message::RecordingError(e.to_string()),
                        }
                    );
                }
                Command::none()
            },

            Message::AudioLevelUpdated(level) => {
                self.audio_level = level;
                Command::none()
            },

            // Session events
            Message::RecordingStateChanged(state) => {
                self.recording_state = state;
                Command::none()
            },

            Message::ElapsedTimeUpdated(duration) => {
                self.elapsed_time = duration;
                Command::none()
            },

            Message::RecordingError(error) => {
                self.error_message = Some(error);
                // Also transition to error state
                self.recording_state = RecordingState::Error(error.clone());
                Command::none()
            },

            // File save
            Message::SaveRecordingPrompt => {
                // Open native file dialog
                Command::perform(
                    async { /* open file dialog */ },
                    Message::SavePathSelected
                )
            },

            Message::SavePathSelected(path) => {
                if let Some(path) = path {
                    self.handle_save_recording(path)
                } else {
                    Command::none()
                }
            },

            // Settings
            Message::UpdateSetting(update) => {
                self.apply_setting_update(update);
                Command::none()
            },

            Message::SaveSettings => {
                let settings = self.settings.clone();
                Command::perform(
                    async move { ConfigStorage::save(&settings) },
                    |result| match result {
                        Ok(_) => Message::Tick,  // Noop
                        Err(e) => Message::RecordingError(e.to_string()),
                    }
                )
            },

            // Periodic updates
            Message::Tick => {
                // Update elapsed time if recording
                if matches!(self.recording_state, RecordingState::Recording) {
                    if let Some(session) = &self.recording_session {
                        self.elapsed_time = session.elapsed_time();
                        self.audio_level = session.audio_level();
                    }
                }
                Command::none()
            },

            // Navigation
            Message::NavigateTo(screen) => {
                self.current_screen = screen;
                Command::none()
            },

            Message::DismissError => {
                self.error_message = None;
                Command::none()
            },

            _ => Command::none(),
        }
    }
}
```

---

## Subscription Protocol

**Purpose**: Define subscriptions for continuous updates

```rust
impl App {
    /// Subscription for periodic updates
    pub fn subscription(&self) -> Subscription<Message> {
        // Update every 16ms (60 fps) for smooth UI
        iced::time::every(Duration::from_millis(16))
            .map(|_| Message::Tick)
    }
}
```

**Subscriptions**:
- **Tick**: Every 16ms for elapsed time and audio level updates
- **Keyboard Events**: Global keyboard shortcuts (platform-specific)

---

## Command Protocol

**Purpose**: Define async commands for side effects

```rust
/// Async command types
pub enum AsyncCommand {
    /// Refresh available screen sources
    RefreshScreenSources,

    /// Refresh camera devices
    RefreshCameras,

    /// Refresh microphone devices
    RefreshMicrophones,

    /// Start recording session
    StartRecording {
        config: RecordingConfig,
    },

    /// Stop recording session
    StopRecording,

    /// Save recording to file
    SaveRecording {
        path: PathBuf,
    },

    /// Load settings from disk
    LoadSettings,

    /// Save settings to disk
    SaveSettings {
        settings: ApplicationSettings,
    },
}
```

**Command Execution**:
- All I/O operations (file dialogs, device enumeration) use `Command::perform()`
- Long-running operations (encoding) spawn background tasks
- Results are sent back as messages

---

## State Transition Validation

**Purpose**: Enforce valid state transitions

```rust
impl App {
    /// Validates if an action is allowed in current state
    fn can_transition(&self, message: &Message) -> bool {
        use Message::*;
        use RecordingState::*;

        match (&self.recording_state, message) {
            // Can only start if idle
            (Idle, StartRecording) => self.selected_source.is_some(),

            // Can only pause if recording
            (Recording, PauseRecording) => true,

            // Can only resume if paused
            (Paused, ResumeRecording) => true,

            // Can stop if recording or paused
            (Recording | Paused, StopRecording) => true,

            // Can cancel from any non-terminal state
            (Idle | Recording | Paused | Stopped, CancelRecording) => true,

            // Can't transition from terminal states
            (Completed | Cancelled | Error(_), StartRecording | PauseRecording | ResumeRecording) => false,

            // Default: allow
            _ => true,
        }
    }

    /// Update wrapper that validates transitions
    pub fn update_safe(&mut self, message: Message) -> Command<Message> {
        if !self.can_transition(&message) {
            let error = format!(
                "Invalid action {:?} in state {:?}",
                message, self.recording_state
            );
            self.error_message = Some(error);
            return Command::none();
        }

        self.update(message)
    }
}
```

---

## Error Message Protocol

**Purpose**: Standardized error display

```rust
/// Error message display contract
pub trait ErrorDisplay {
    /// Returns user-friendly error message
    fn user_message(&self) -> String;

    /// Returns actionable recovery steps
    fn recovery_steps(&self) -> Vec<String>;

    /// Returns severity level
    fn severity(&self) -> ErrorSeverity;
}

#[derive(Debug, Clone, Copy, PartialEq)]
pub enum ErrorSeverity {
    Info,      // Informational (e.g., device disconnected but can continue)
    Warning,   // Warning (e.g., low disk space)
    Error,     // Error (recording failed)
    Critical,  // Critical (app must close)
}

impl ErrorDisplay for ScreenCaptureError {
    fn user_message(&self) -> String {
        match self {
            Self::PermissionDenied { platform_instructions } => {
                format!("Screen recording permission required. {}", platform_instructions)
            },
            Self::SourceUnavailable(reason) => {
                format!("Screen source is no longer available: {}", reason)
            },
            Self::CaptureFailed(reason) => {
                format!("Screen capture failed: {}", reason)
            },
            Self::InvalidSource => {
                "Invalid screen source selected".to_string()
            },
        }
    }

    fn recovery_steps(&self) -> Vec<String> {
        match self {
            Self::PermissionDenied { .. } => vec![
                "Grant screen recording permission in system settings".to_string(),
                "Restart the application".to_string(),
            ],
            Self::SourceUnavailable(_) => vec![
                "Select a different screen or window".to_string(),
            ],
            Self::CaptureFailed(_) => vec![
                "Try restarting the recording".to_string(),
                "Check if another app is using screen capture".to_string(),
            ],
            Self::InvalidSource => vec![
                "Select a valid screen or window from the list".to_string(),
            ],
        }
    }

    fn severity(&self) -> ErrorSeverity {
        match self {
            Self::PermissionDenied { .. } => ErrorSeverity::Error,
            Self::SourceUnavailable(_) => ErrorSeverity::Warning,
            Self::CaptureFailed(_) => ErrorSeverity::Error,
            Self::InvalidSource => ErrorSeverity::Warning,
        }
    }
}
```

---

## UI Component Messages

**Purpose**: Messages scoped to specific UI components

```rust
/// Messages for camera overlay widget
#[derive(Debug, Clone)]
pub enum CameraOverlayMessage {
    StartDrag,
    Drag { x: f32, y: f32 },
    EndDrag,
    Resize { width: u32, height: u32 },
    ChangeShape(OverlayShape),
}

/// Messages for audio level indicator widget
#[derive(Debug, Clone)]
pub enum AudioMeterMessage {
    LevelChanged(f32),
    Muted(bool),
}

/// Messages for recording timer widget
#[derive(Debug, Clone)]
pub enum TimerMessage {
    TimeUpdated(Duration),
    FormatChanged(TimerFormat),
}

#[derive(Debug, Clone, Copy)]
pub enum TimerFormat {
    MMSS,      // Minutes:Seconds (e.g., 05:42)
    HHMMSS,    // Hours:Minutes:Seconds (e.g., 01:05:42)
}
```

These component messages are mapped to main `Message` enum in parent widgets.

---

## Performance Contract

**Requirements**:
- ✅ Message processing MUST complete in < 16ms (60 fps UI)
- ✅ Tick subscription MUST update UI elements within 16ms
- ✅ Heavy operations (device enumeration, file save) MUST use async commands
- ✅ UI MUST remain responsive during recording (no blocking operations on main thread)

---

## Thread Safety

- **Main Thread**: Handles all UI messages and state updates (iced runtime)
- **Background Threads**: Recording session runs on separate thread, sends messages back via channels
- **Message Queue**: Thread-safe message passing between recording thread and UI thread

---

## Summary

This protocol ensures:
- ✅ **Type safety**: All user interactions are typed messages
- ✅ **State management**: Centralized state with validated transitions
- ✅ **Async operations**: Non-blocking I/O via commands
- ✅ **Error handling**: Consistent error display with recovery steps
- ✅ **Performance**: 60 fps UI with periodic updates via subscriptions
- ✅ **Testability**: Pure functions for state updates (easy to test)
