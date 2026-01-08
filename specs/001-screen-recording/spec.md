# Feature Specification: Screen Recording with Camera Overlay

**Feature Branch**: `001-screen-recording`
**Created**: 2026-01-02
**Status**: Draft
**Input**: User description: "Build a desktop application to record a screen, capture a camera to display on top of the screen recording and audio input. It's a loom alternative. I must be able to record a screen with my camera view on top of the screen and in the end save the video on my disk."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Basic Screen Recording (Priority: P1)

As a user, I want to record my screen and save it as a video file on my local disk, so I can capture tutorials, demos, or bug reports without relying on cloud services.

**Why this priority**: This is the core MVP functionality. Without the ability to record and save the screen, the application has no value. This establishes the foundation for all other features.

**Independent Test**: Can be fully tested by launching the app, selecting a screen/window to record, starting recording, stopping recording, and verifying the video file is saved to disk and playable.

**Acceptance Scenarios**:

1. **Given** the application is launched, **When** I select "Record Screen" and choose my entire screen, **Then** the recording starts and a recording indicator is visible
2. **Given** a screen recording is in progress, **When** I click "Stop Recording", **Then** the recording stops and I am prompted to choose a save location
3. **Given** I have stopped a recording and selected a save location, **When** the save completes, **Then** a video file exists at that location and is playable in standard video players
4. **Given** the application is launched, **When** I select "Record Screen" and choose a specific window, **Then** only that window is recorded (not the entire screen)
5. **Given** I am recording my screen, **When** I switch to a different window or application, **Then** the recording continues uninterrupted

---

### User Story 2 - Camera Overlay (Priority: P2)

As a user, I want to display my camera feed as an overlay on top of my screen recording, so I can create personalized video tutorials showing both my screen and my face.

**Why this priority**: Camera overlay is the key differentiator from basic screen recording tools and matches the Loom-like experience. However, screen recording alone is still useful, making this a P2 feature that enhances the core functionality.

**Independent Test**: Can be fully tested by starting a recording with camera enabled, verifying the camera feed appears as an overlay during recording, and confirming the saved video includes the camera overlay.

**Acceptance Scenarios**:

1. **Given** the application is launched, **When** I enable "Show Camera" before starting a recording, **Then** I see a preview of my camera feed
2. **Given** camera is enabled and I start recording, **When** the recording is in progress, **Then** my camera feed appears as a movable overlay on the screen recording
3. **Given** a recording is in progress with camera overlay, **When** I drag the camera overlay to a different position, **Then** the overlay moves to the new position and records in that location
4. **Given** a recording is in progress with camera overlay, **When** I resize the camera overlay, **Then** the camera feed scales appropriately and records at the new size
5. **Given** I have completed a recording with camera enabled, **When** I play back the saved video, **Then** the camera overlay is visible at the position and size I configured during recording
6. **Given** camera is enabled, **When** no camera device is detected or camera access is denied, **Then** I see an actionable error message explaining how to grant camera access

---

### User Story 3 - Audio Recording (Priority: P2)

As a user, I want to record audio from my microphone along with my screen, so I can narrate my screen recordings and create complete video tutorials.

**Why this priority**: Audio narration is essential for tutorials and demos, making recordings much more valuable. Like camera overlay, it enhances the core screen recording but isn't strictly required for basic functionality.

**Independent Test**: Can be fully tested by starting a recording with microphone enabled, speaking during the recording, and verifying the saved video includes synchronized audio.

**Acceptance Scenarios**:

1. **Given** the application is launched, **When** I enable "Record Microphone" before starting a recording, **Then** I see a microphone level indicator showing my audio input
2. **Given** microphone is enabled and I start recording, **When** I speak into my microphone during recording, **Then** the audio level indicator shows activity
3. **Given** I have completed a recording with microphone enabled, **When** I play back the saved video, **Then** my voice is audible and synchronized with the screen recording
4. **Given** microphone is enabled, **When** no microphone device is detected or microphone access is denied, **Then** I see an actionable error message explaining how to grant microphone access
5. **Given** multiple microphone devices are available, **When** I open audio settings, **Then** I can select which microphone to use for recording

---

### User Story 4 - Recording Controls (Priority: P3)

As a user, I want to pause and resume recordings, use keyboard shortcuts, and see recording time, so I can efficiently create recordings without awkward interruptions or having to edit out mistakes.

**Why this priority**: These convenience features improve the recording experience but are not essential for the MVP. Users can work around the lack of pause by stopping and creating multiple recordings, and keyboard shortcuts are nice-to-have.

**Independent Test**: Can be fully tested by starting a recording, using pause/resume and keyboard shortcuts, and verifying the recording reflects only the non-paused portions.

**Acceptance Scenarios**:

1. **Given** a recording is in progress, **When** I click "Pause", **Then** the recording pauses and the timer stops incrementing
2. **Given** a recording is paused, **When** I click "Resume", **Then** the recording continues from where it paused (no gap in the final video)
3. **Given** the application is running, **When** I press a configurable keyboard shortcut, **Then** recording starts/stops without needing to click the UI
4. **Given** a recording is in progress, **When** I look at the recording indicator, **Then** I see the elapsed recording time in MM:SS format
5. **Given** I have paused and resumed a recording multiple times, **When** I save the final video, **Then** only the non-paused segments are included (no frozen frames during paused periods)

---

### Edge Cases

- What happens when the user runs out of disk space during recording?
- What happens if the selected window is closed during recording?
- What happens if the camera or microphone is disconnected during an active recording?
- What happens if the user tries to start a new recording while one is already in progress?
- What happens if the system goes to sleep or the user locks the screen during recording?
- What happens if the user denies screen recording permissions on systems that require it (macOS)?
- What happens if the application crashes during an active recording?
- What happens if the user tries to record a protected/DRM content window?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to select between recording the entire screen or a specific window
- **FR-002**: System MUST capture screen content at a minimum of 30 frames per second for smooth playback
- **FR-003**: System MUST provide visual feedback when recording is active (recording indicator, timer, etc.)
- **FR-004**: System MUST allow users to stop an active recording
- **FR-005**: System MUST prompt users to select a save location when recording is stopped
- **FR-006**: System MUST save recordings as video files in a widely-supported format (H.264 MP4 or similar)
- **FR-007**: System MUST allow users to enable/disable camera overlay before starting a recording
- **FR-008**: System MUST display a live preview of the camera feed when camera is enabled
- **FR-009**: System MUST allow users to reposition and resize the camera overlay during recording
- **FR-010**: System MUST allow users to enable/disable microphone recording before starting a recording
- **FR-011**: System MUST display a microphone level indicator when microphone is enabled
- **FR-012**: System MUST synchronize audio and video tracks in the final saved recording
- **FR-013**: System MUST request necessary system permissions (screen recording, camera, microphone) and provide clear feedback if denied
- **FR-014**: System MUST handle errors gracefully (disk full, device disconnected, permission denied) with actionable error messages
- **FR-015**: System MUST allow users to select which microphone and camera device to use when multiple are available
- **FR-016**: System MUST work on macOS, Linux, and Windows operating systems
- **FR-017**: System MUST save user preferences (default save location, camera position/size, selected devices) between sessions
- **FR-018**: System MUST prevent data loss if the application crashes during recording by attempting to save the recorded content up to the crash point

### Key Entities

- **Recording Session**: Represents an active or completed recording, including configuration (screen/window, camera enabled, microphone enabled), recording state (recording/paused/stopped), elapsed time, and output file path
- **Camera Overlay**: Represents the camera feed configuration including device selection, position (x, y coordinates), size (width, height), and visibility state
- **Audio Input**: Represents the microphone configuration including device selection, input level, and enabled state
- **Recording Output**: Represents the saved video file including file path, format, resolution, frame rate, duration, and file size
- **Application Settings**: Represents user preferences including default save location, default camera settings, default microphone settings, keyboard shortcuts, and preferred output format/quality

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can complete a basic screen recording (select screen, record, stop, save) in under 30 seconds for their first recording
- **SC-002**: Recordings are saved successfully 99.9% of the time when sufficient disk space is available and permissions are granted
- **SC-003**: Recorded video playback quality is indistinguishable from real-time screen content for text and UI elements (no visible compression artifacts on standard content)
- **SC-004**: Audio and video synchronization drift is less than 100ms over a 10-minute recording
- **SC-005**: Application launches in under 2 seconds on modern mid-range hardware (per constitution performance requirements)
- **SC-006**: Recording overhead causes less than 5% CPU usage increase on target hardware during typical screen recording (per constitution performance requirements)
- **SC-007**: Users can successfully start a recording with camera and microphone on first attempt 90% of the time (after granting initial permissions)
- **SC-008**: Saved video files are playable in all major video players (VLC, QuickTime, Windows Media Player, mpv)
- **SC-009**: Camera overlay repositioning responds within 100ms of user drag action
- **SC-010**: All recordings are stored locally on user's machine with no data transmitted to external servers

### Assumptions

- Users have modern desktop/laptop hardware with at least 8GB RAM and multi-core processors (typical consumer hardware from 2018+)
- Users have sufficient disk space for their recordings (minimum 1GB free space for typical 10-minute recording)
- Users are familiar with basic desktop application concepts (buttons, file dialogs, permissions)
- Video output format will default to H.264 MP4 for maximum compatibility, with quality configurable by user
- Default video resolution will match the recorded screen/window resolution (no upscaling or downscaling unless specified)
- Camera overlay will default to a circular or rounded rectangle shape positioned in the bottom-right corner
- Keyboard shortcuts will follow platform conventions (Cmd on macOS, Ctrl on Windows/Linux)
- Application will use platform-native file dialogs and permission requests
