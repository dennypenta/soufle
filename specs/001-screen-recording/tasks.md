# Tasks: Screen Recording with Camera Overlay

**Input**: Design documents from `/specs/001-screen-recording/`
**Prerequisites**: plan.md, spec.md, data-model.md, contracts/, research.md, quickstart.md

**Tests**: Tests are OPTIONAL per specification. Not generating test tasks unless explicitly requested by user.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story (P1→P2→P3).

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Single project**: `src/`, `tests/` at repository root
- All paths shown below use single project structure per plan.md

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Initialize Cargo.toml with all dependencies from research.md (iced 0.12, xcap 0.0.9, ffmpeg-next 7.0, nokhwa 0.10, cpal 0.15, serde, toml, dirs, thiserror, tracing, uuid, criterion)
- [ ] T002 Create rustfmt.toml with edition=2021, max_width=100
- [ ] T003 Create .clippy.toml with deny warnings configuration
- [ ] T004 [P] Create src/main.rs with tracing initialization
- [ ] T005 [P] Create src/error.rs with SoufleError enum using thiserror
- [ ] T006 [P] Create src/ui/mod.rs module file
- [ ] T007 [P] Create src/recording/mod.rs module file
- [ ] T008 [P] Create src/platform/mod.rs module file
- [ ] T009 [P] Create src/config/mod.rs module file
- [ ] T010 [P] Create tests/integration/mod.rs directory structure
- [ ] T011 [P] Create benches/recording.rs with criterion setup
- [ ] T012 Configure Cargo.toml profile.release with lto=fat, codegen-units=1 per constitution

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T013 Implement ScreenCapturer trait in src/recording/screen.rs with capture_frame, dimensions, is_valid, current_fps methods
- [ ] T014 Implement CapturedFrame struct in src/recording/screen.rs with data, width, height, timestamp fields
- [ ] T015 Implement VideoEncoder trait in src/recording/encoder.rs with encode_frame, finalize methods
- [ ] T016 Implement RecordingState enum in src/recording/session.rs with Idle, Recording, Paused, Stopped, Saving, Completed, Cancelled, Error variants
- [ ] T017 Implement ScreenSource enum in src/recording/screen.rs with EntireScreen and SpecificWindow variants
- [ ] T018 [P] Create platform-specific module stubs in src/platform/macos.rs, src/platform/linux.rs, src/platform/windows.rs with #[cfg(target_os)] attributes
- [ ] T019 Implement ApplicationSettings struct in src/config/settings.rs with default_save_directory, default_format, default_codec, keyboard_shortcuts fields
- [ ] T020 Implement ConfigStorage trait in src/config/storage.rs with load, save, reset_to_defaults methods using TOML serialization

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Basic Screen Recording (Priority: P1) 🎯 MVP

**Goal**: Record entire screen or specific window and save as MP4 file to local disk

**Independent Test**: Launch app → select screen/window → start recording → stop recording → save to file → verify MP4 is playable in VLC/QuickTime/Windows Media Player

### Implementation for User Story 1

- [ ] T021 [P] [US1] Implement XCapScreenCapturer struct in src/recording/screen.rs wrapping xcap library for cross-platform screen capture
- [ ] T022 [P] [US1] Implement screen source enumeration functions (list_displays, list_windows) in src/recording/screen.rs using xcap APIs
- [ ] T023 [US1] Implement FfmpegEncoder struct in src/recording/encoder.rs with H.264 codec and MP4 muxing using ffmpeg-next
- [ ] T024 [US1] Add frame encoding logic to FfmpegEncoder::encode_frame in src/recording/encoder.rs (convert RGBA to YUV, encode with libx264)
- [ ] T025 [US1] Add finalization logic to FfmpegEncoder::finalize in src/recording/encoder.rs (close encoder, write MP4 trailer)
- [ ] T026 [US1] Implement RecordingSession struct in src/recording/session.rs with id, state, start_time, elapsed_time, pause_segments, screen_source, output_path fields
- [ ] T027 [US1] Implement RecordingSession::start method in src/recording/session.rs to transition from Idle to Recording and spawn capture loop
- [ ] T028 [US1] Implement RecordingSession::stop method in src/recording/session.rs to transition to Stopped state
- [ ] T029 [US1] Implement RecordingSession::save method in src/recording/session.rs to encode captured frames and write to output_path
- [ ] T030 [US1] Implement RecordingSession::elapsed_time method in src/recording/session.rs to calculate duration excluding pause segments
- [ ] T031 [US1] Create App struct in src/ui/app.rs with recording_session, recording_state, selected_source fields implementing iced::Application trait
- [ ] T032 [US1] Define Message enum in src/ui/app.rs with StartRecording, StopRecording, SelectDisplay, SelectWindow, SaveRecordingPrompt, SavePathSelected, RecordingError variants
- [ ] T033 [US1] Implement App::new in src/ui/app.rs to initialize default application state
- [ ] T034 [US1] Implement App::update in src/ui/app.rs to handle StartRecording, StopRecording, SaveRecordingPrompt, SavePathSelected messages
- [ ] T035 [US1] Implement App::view in src/ui/app.rs with screen source selection dropdown, Start/Stop buttons, and file save dialog
- [ ] T036 [US1] Create home screen UI in src/ui/screens/home.rs with recording controls layout (screen selector, start button, stop button)
- [ ] T037 [US1] Implement screen source selection widget in src/ui/screens/home.rs using iced::widget::pick_list for displays and windows
- [ ] T038 [US1] Add visual recording indicator to src/ui/screens/home.rs (red dot or recording icon visible during Recording state)
- [ ] T039 [US1] Integrate native file dialog for save path selection in src/ui/app.rs using rfd crate or iced file dialog
- [ ] T040 [US1] Add error display widget in src/ui/screens/home.rs to show actionable error messages from RecordingError variant
- [ ] T041 [US1] Update src/main.rs to launch iced::Application with App::run(Settings::default())
- [ ] T042 [US1] Add disk space validation in src/recording/encoder.rs::check_disk_space before encoding (return InsufficientDiskSpace error if needed)
- [ ] T043 [US1] Add permission error handling in src/recording/screen.rs for screen capture permissions (platform-specific instructions in error message)

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently. You can launch the app, record your screen, and save a playable MP4 video.

---

## Phase 4: User Story 2 - Camera Overlay (Priority: P2)

**Goal**: Display camera feed as movable/resizable overlay on screen recording

**Independent Test**: Start recording with camera enabled → verify camera preview appears → drag overlay to new position → resize overlay → stop recording → playback shows camera overlay at final position/size

### Implementation for User Story 2

- [ ] T044 [P] [US2] Implement CameraCapturer trait in src/recording/camera.rs with capture_frame, resolution, is_available methods
- [ ] T045 [P] [US2] Implement CameraFrame struct in src/recording/camera.rs with data (RGB), width, height, timestamp fields
- [ ] T046 [P] [US2] Implement CameraDevice struct in src/recording/camera.rs with id, name, resolutions fields
- [ ] T047 [US2] Implement NokhwaCamera struct in src/recording/camera.rs wrapping nokhwa library for cross-platform camera access
- [ ] T048 [US2] Add camera device enumeration function (list_devices) in src/recording/camera.rs using nokhwa APIs
- [ ] T049 [US2] Implement camera frame capture in NokhwaCamera::capture_frame in src/recording/camera.rs with format conversion to RGB
- [ ] T050 [US2] Implement CameraOverlay struct in src/config/settings.rs with device_id, enabled, position, size, shape fields
- [ ] T051 [US2] Add Position and Size types in src/config/settings.rs (Position with normalized x/y 0.0-1.0, Size with pixel width/height)
- [ ] T052 [US2] Implement OverlayShape enum in src/config/settings.rs with Circle, Rectangle, RoundedRect variants
- [ ] T053 [US2] Update RecordingSession in src/recording/session.rs to include optional camera_capturer field
- [ ] T054 [US2] Update FfmpegEncoder::encode_frame in src/recording/encoder.rs to composite camera overlay onto screen frame before encoding
- [ ] T055 [US2] Implement overlay compositing logic in src/recording/encoder.rs (blend camera RGB onto screen RGBA at specified position/size)
- [ ] T056 [US2] Add circular mask rendering for Circle overlay shape in src/recording/encoder.rs
- [ ] T057 [US2] Update App struct in src/ui/app.rs to include camera_enabled, available_cameras, selected_camera, camera_overlay fields
- [ ] T058 [US2] Add camera control messages to Message enum in src/ui/app.rs (ToggleCamera, RefreshCameras, CamerasRefreshed, SelectCamera, MoveCameraOverlay, ResizeCameraOverlay, ChangeCameraShape)
- [ ] T059 [US2] Implement ToggleCamera message handler in App::update in src/ui/app.rs to load cameras and enable preview
- [ ] T060 [US2] Create camera overlay widget in src/ui/widgets/camera_overlay.rs with drag/resize functionality using iced canvas or custom widget
- [ ] T061 [US2] Implement camera preview display in src/ui/widgets/camera_overlay.rs showing live camera feed before recording
- [ ] T062 [US2] Add camera device selector dropdown in src/ui/screens/home.rs using iced::widget::pick_list
- [ ] T063 [US2] Add camera overlay position/size controls in src/ui/screens/home.rs (drag handles, resize corners)
- [ ] T064 [US2] Add camera shape selector in src/ui/screens/home.rs (Circle, Rectangle, RoundedRect radio buttons)
- [ ] T065 [US2] Integrate camera overlay widget into App::view in src/ui/app.rs to show live preview and controls
- [ ] T066 [US2] Add camera permission error handling in src/recording/camera.rs with platform-specific instructions
- [ ] T067 [US2] Add camera disconnection detection in src/recording/camera.rs::is_available method

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently. You can record with or without camera overlay.

---

## Phase 5: User Story 3 - Audio Recording (Priority: P2)

**Goal**: Capture microphone audio and synchronize with video in final MP4

**Independent Test**: Start recording with microphone enabled → speak during recording → stop recording → playback shows synchronized audio with <100ms drift

### Implementation for User Story 3

- [ ] T068 [P] [US3] Implement AudioCapturer trait in src/recording/audio.rs with start, stop, drain_samples, current_level methods
- [ ] T069 [P] [US3] Implement AudioConfig struct in src/recording/audio.rs with sample_rate (44100/48000), channels (1/2) fields
- [ ] T070 [P] [US3] Implement AudioDevice struct in src/recording/audio.rs with id, name, supported_configs fields
- [ ] T071 [US3] Implement CpalAudio struct in src/recording/audio.rs wrapping cpal library for cross-platform audio capture
- [ ] T072 [US3] Add audio device enumeration function (list_devices) in src/recording/audio.rs using cpal host API
- [ ] T073 [US3] Implement audio stream capture in CpalAudio::start in src/recording/audio.rs with callback-based sample buffer
- [ ] T074 [US3] Implement RMS level calculation in CpalAudio::current_level in src/recording/audio.rs for real-time metering
- [ ] T075 [US3] Add audio sample buffer with Arc<Mutex<Vec<f32>>> in src/recording/audio.rs for thread-safe access
- [ ] T076 [US3] Update RecordingSession in src/recording/session.rs to include optional audio_capturer field
- [ ] T077 [US3] Update FfmpegEncoder in src/recording/encoder.rs to add encode_audio method accepting f32 samples
- [ ] T078 [US3] Implement audio encoding in FfmpegEncoder::encode_audio in src/recording/encoder.rs using AAC codec
- [ ] T079 [US3] Add audio/video muxing logic in FfmpegEncoder::finalize in src/recording/encoder.rs to synchronize streams
- [ ] T080 [US3] Implement timestamp-based A/V sync in src/recording/encoder.rs using CapturedFrame and audio buffer timestamps
- [ ] T081 [US3] Update App struct in src/ui/app.rs to include microphone_enabled, available_microphones, selected_microphone, audio_level fields
- [ ] T082 [US3] Add audio control messages to Message enum in src/ui/app.rs (ToggleMicrophone, RefreshMicrophones, MicrophonesRefreshed, SelectMicrophone, AudioLevelUpdated)
- [ ] T083 [US3] Implement ToggleMicrophone message handler in App::update in src/ui/app.rs to start audio capture
- [ ] T084 [US3] Create audio level indicator widget in src/ui/widgets/indicators.rs with progress bar or VU meter showing audio_level 0.0-1.0
- [ ] T085 [US3] Add microphone device selector dropdown in src/ui/screens/home.rs using iced::widget::pick_list
- [ ] T086 [US3] Integrate audio level indicator into App::view in src/ui/app.rs below microphone selector
- [ ] T087 [US3] Add subscription for periodic audio level updates in App::subscription in src/ui/app.rs using iced::time::every (16ms for 60fps)
- [ ] T088 [US3] Add microphone permission error handling in src/recording/audio.rs with platform-specific instructions
- [ ] T089 [US3] Add microphone disconnection detection in src/recording/audio.rs::is_available method

**Checkpoint**: All user stories 1-3 should now be independently functional. Full feature set: screen + camera + audio recording.

---

## Phase 6: User Story 4 - Recording Controls (Priority: P3)

**Goal**: Add pause/resume, keyboard shortcuts, and recording timer

**Independent Test**: Start recording → pause → resume → pause → resume → stop → verify final video excludes paused segments (no frozen frames)

### Implementation for User Story 4

- [ ] T090 [US4] Update RecordingState enum in src/recording/session.rs to ensure Paused state is handled in all state transitions
- [ ] T091 [US4] Implement RecordingSession::pause method in src/recording/session.rs to transition from Recording to Paused and record pause start time
- [ ] T092 [US4] Implement RecordingSession::resume method in src/recording/session.rs to transition from Paused to Recording and record pause end time
- [ ] T093 [US4] Update RecordingSession::elapsed_time in src/recording/session.rs to exclude all pause_segments from total duration
- [ ] T094 [US4] Update FfmpegEncoder in src/recording/encoder.rs to skip frames during pause segments (check timestamp against pause ranges)
- [ ] T095 [US4] Add PauseRecording, ResumeRecording messages to Message enum in src/ui/app.rs
- [ ] T096 [US4] Implement pause/resume message handlers in App::update in src/ui/app.rs calling session.pause() and session.resume()
- [ ] T097 [US4] Create recording timer widget in src/ui/widgets/indicators.rs displaying elapsed_time in MM:SS format
- [ ] T098 [US4] Add Pause and Resume buttons to src/ui/screens/home.rs (visible only when appropriate state)
- [ ] T099 [US4] Integrate recording timer widget into App::view in src/ui/app.rs near recording controls
- [ ] T100 [US4] Add keyboard shortcut configuration to ApplicationSettings in src/config/settings.rs with HashMap<Action, KeyBinding>
- [ ] T101 [US4] Define Action enum in src/config/settings.rs with StartRecording, StopRecording, PauseRecording, ResumeRecording, ToggleCamera, ToggleMicrophone variants
- [ ] T102 [US4] Define KeyBinding struct in src/config/settings.rs with key, modifiers (ctrl, shift, alt, cmd) fields
- [ ] T103 [US4] Implement default keyboard shortcuts in ApplicationSettings::default in src/config/settings.rs (platform-specific: Cmd on macOS, Ctrl on Windows/Linux)
- [ ] T104 [US4] Add keyboard event subscription in App::subscription in src/ui/app.rs using iced::keyboard::Event listener
- [ ] T105 [US4] Implement keyboard shortcut matching in App::update in src/ui/app.rs to trigger messages based on KeyBinding configuration
- [ ] T106 [US4] Create settings screen in src/ui/screens/settings.rs with keyboard shortcut configuration UI
- [ ] T107 [US4] Add OpenSettings, CloseSettings messages to Message enum in src/ui/app.rs
- [ ] T108 [US4] Implement settings screen navigation in App::view in src/ui/app.rs with settings button and modal/screen transition

**Checkpoint**: All user stories should now be complete with full feature parity to requirements.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T109 [P] Add comprehensive error messages with platform-specific instructions to all error types in src/error.rs (PermissionDenied, DeviceNotFound, InsufficientDiskSpace)
- [ ] T110 [P] Implement ErrorDisplay trait in src/error.rs with user_message, recovery_steps, severity methods for all error types
- [ ] T111 [P] Add state transition validation in App::update_safe in src/ui/app.rs to prevent invalid transitions (e.g., Pause from Idle)
- [ ] T112 [P] Implement ConfigStorage::load in src/config/storage.rs to read TOML from platform-specific directory (XDG/AppData/Application Support)
- [ ] T113 [P] Implement ConfigStorage::save in src/config/storage.rs to write ApplicationSettings as TOML
- [ ] T114 [P] Add settings persistence on app close in src/main.rs or App drop handler
- [ ] T115 [P] Add settings loading on app launch in App::new in src/ui/app.rs
- [ ] T116 [P] Create platform-specific Info.plist for macOS in project root with NSCameraUsageDescription, NSMicrophoneUsageDescription, NSScreenCaptureUsageDescription
- [ ] T117 Add logging throughout recording pipeline in src/recording/ modules using tracing::info, tracing::warn, tracing::error (no println! or dbg!)
- [ ] T118 Add performance profiling annotations in critical paths using tracing::span for screen capture, encoding, A/V sync
- [ ] T119 Implement graceful shutdown in src/recording/session.rs to save in-progress recordings on crash (attempt recovery in RecordingSession drop)
- [ ] T120 Add input validation for all user-provided paths in src/config/storage.rs using Path::canonicalize to prevent directory traversal
- [ ] T121 Create integration test in tests/integration/recording_flow.rs for complete P1 workflow (start → record → stop → save → verify playable)
- [ ] T122 Create integration test in tests/integration/camera_overlay.rs for P2 workflow with camera enabled
- [ ] T123 Create integration test in tests/integration/audio_sync.rs to verify A/V sync drift <100ms over 10-minute recording
- [ ] T124 Create performance benchmark in benches/recording.rs to verify screen capture <5% CPU overhead at 30fps
- [ ] T125 Create performance benchmark in benches/recording.rs to verify video encoding ≥2x real-time speed
- [ ] T126 Create performance benchmark in benches/recording.rs to verify <2s application startup time
- [ ] T127 Run cargo clippy -- -D warnings and fix all warnings per constitution
- [ ] T128 Run cargo fmt and verify code formatting per constitution
- [ ] T129 Run cargo audit and ensure no dependency vulnerabilities
- [ ] T130 Add dependency justification comments in Cargo.toml per constitution

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational phase completion - No dependencies on other stories ✅ MVP
- **User Story 2 (Phase 4)**: Depends on Foundational phase completion - Can run in parallel with US3/US4 if staffed
- **User Story 3 (Phase 5)**: Depends on Foundational phase completion - Can run in parallel with US2/US4 if staffed
- **User Story 4 (Phase 6)**: Depends on User Story 1 (pause/resume requires basic recording session) - Should wait for US1 completion
- **Polish (Phase 7)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1) - Basic Screen Recording**: Can start after Foundational (Phase 2) ✅ **FULLY INDEPENDENT** ✅ **MVP READY**
- **User Story 2 (P2) - Camera Overlay**: Can start after Foundational (Phase 2) - Extends US1 but independently testable
- **User Story 3 (P2) - Audio Recording**: Can start after Foundational (Phase 2) - Extends US1 but independently testable
- **User Story 4 (P3) - Recording Controls**: Requires US1 complete (builds on RecordingSession from US1)

### Within Each User Story

**User Story 1**:
- T021-T022 (screen capture setup) can run in parallel [P]
- T023-T025 (encoder implementation) must run sequentially
- T026-T030 (session management) must run after T023-T025
- T031-T040 (UI implementation) can start in parallel with T026-T030
- T041-T043 (integration) must run last

**User Story 2**:
- T044-T046 (camera interfaces) can run in parallel [P]
- T047-T049 (camera implementation) must run after interfaces
- T050-T052 (overlay config) can run in parallel with camera [P]
- T053-T057 (encoder/session updates) must run sequentially
- T058-T067 (UI implementation) can run in parallel after interfaces complete

**User Story 3**:
- T068-T070 (audio interfaces) can run in parallel [P]
- T071-T075 (audio implementation) must run after interfaces
- T076-T080 (encoder integration) must run sequentially
- T081-T089 (UI implementation) can run in parallel after interfaces complete

**User Story 4**:
- T090-T094 (pause/resume logic) must run sequentially
- T095-T099 (pause UI) can run in parallel with T100-T108 (shortcuts) [P]
- T100-T105 (shortcut implementation) can run in parallel [P]
- T106-T108 (settings screen) must run last

### Parallel Opportunities

**Setup Phase (Phase 1)**:
```bash
# Launch T004-T011 in parallel:
- T004: src/main.rs
- T005: src/error.rs
- T006: src/ui/mod.rs
- T007: src/recording/mod.rs
- T008: src/platform/mod.rs
- T009: src/config/mod.rs
- T010: tests/integration/mod.rs
- T011: benches/recording.rs
```

**User Story 1 - Models**:
```bash
# After T020, launch T021-T022 in parallel:
- T021: XCapScreenCapturer
- T022: screen enumeration
```

**User Story 2 - Interfaces**:
```bash
# Launch T044-T046 in parallel:
- T044: CameraCapturer trait
- T045: CameraFrame struct
- T046: CameraDevice struct
```

**User Story 3 - Interfaces**:
```bash
# Launch T068-T070 in parallel:
- T068: AudioCapturer trait
- T069: AudioConfig struct
- T070: AudioDevice struct
```

**Polish Phase**:
```bash
# Launch T109-T116 in parallel (different files):
- T109: Error messages
- T110: ErrorDisplay trait
- T111: State validation
- T112: Config load
- T113: Config save
- T114: Settings persistence
- T115: Settings loading
- T116: Info.plist
```

---

## Implementation Strategy

### MVP First (User Story 1 Only) - Recommended

1. Complete Phase 1: Setup (T001-T012)
2. Complete Phase 2: Foundational (T013-T020) ✅ **CRITICAL GATE**
3. Complete Phase 3: User Story 1 (T021-T043)
4. **STOP and VALIDATE**: Test US1 independently:
   - Launch app ✅
   - Select screen/window ✅
   - Start recording ✅
   - Stop recording ✅
   - Save video ✅
   - Verify playable in VLC ✅
5. If working, deploy/demo MVP

### Incremental Delivery (Recommended)

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 (T021-T043) → Test independently → **Deploy/Demo MVP** ✅
3. Add User Story 2 (T044-T067) → Test independently → Deploy/Demo (MVP + Camera)
4. Add User Story 3 (T068-T089) → Test independently → Deploy/Demo (MVP + Camera + Audio)
5. Add User Story 4 (T090-T108) → Test independently → Deploy/Demo (Full feature set)
6. Polish (T109-T130) → Final release

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (T001-T020)
2. Once Foundational is done (after T020):
   - **Developer A**: User Story 1 (T021-T043) ✅ MVP priority
   - **Developer B**: User Story 2 (T044-T067)
   - **Developer C**: User Story 3 (T068-T089)
3. After US1 complete:
   - **Developer A**: User Story 4 (T090-T108) - depends on US1
4. Polish (T109-T130) - whole team in parallel

---

## Notes

- [P] tasks = different files, no dependencies - can launch in parallel
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Stop at any checkpoint to validate story independently
- Commit after each task or logical group
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence

---

## Task Summary

**Total Tasks**: 130
- **Setup**: 12 tasks (T001-T012)
- **Foundational**: 8 tasks (T013-T020) ⚠️ **BLOCKS ALL STORIES**
- **User Story 1 (P1 - MVP)**: 23 tasks (T021-T043) 🎯
- **User Story 2 (P2)**: 24 tasks (T044-T067)
- **User Story 3 (P2)**: 22 tasks (T068-T089)
- **User Story 4 (P3)**: 19 tasks (T090-T108)
- **Polish**: 22 tasks (T109-T130)

**Parallel Opportunities**: 45 tasks marked [P] can run in parallel with other tasks in same phase

**MVP Scope (Recommended First Delivery)**:
- Phase 1: Setup (12 tasks)
- Phase 2: Foundational (8 tasks)
- Phase 3: User Story 1 (23 tasks)
- **Total for MVP: 43 tasks** ✅

**Independent Test Criteria**:
- ✅ **US1**: Launch → select screen → record → stop → save → play in VLC
- ✅ **US2**: Enable camera → preview shows → drag overlay → record → play shows overlay
- ✅ **US3**: Enable mic → see level meter → record → play shows audio synced
- ✅ **US4**: Start → pause → resume → stop → play excludes paused segments
