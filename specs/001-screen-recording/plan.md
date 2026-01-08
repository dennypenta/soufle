# Implementation Plan: Screen Recording with Camera Overlay

**Branch**: `001-screen-recording` | **Date**: 2026-01-02 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-screen-recording/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Build a cross-platform desktop application (macOS, Linux, Windows) that records screen content with optional camera overlay and audio narration, saving videos locally to disk. Core MVP focuses on basic screen recording with file save functionality. Enhanced features include camera overlay (movable/resizable) and microphone audio capture. All processing happens locally with no cloud dependencies, ensuring user privacy and data ownership.

## Technical Context

**Language/Version**: Rust 1.75+ (stable channel, 2021 edition)
**Primary Dependencies**: iced 0.12+ (GUI), xcap 0.0.9+ (screen capture), ffmpeg-next 7.0+ (video encoding/muxing), nokhwa 0.10+ (camera capture), cpal 0.15+ (audio capture), serde + toml (config), thiserror (errors), tracing (logging)
**Storage**: File system only (no database - recordings saved as video files, preferences in TOML config file)
**Testing**: cargo test (unit/integration), cargo-tarpaulin (coverage), criterion (benchmarks)
**Target Platform**: Desktop - macOS 10.15+, Linux (X11/Wayland), Windows 10+
**Project Type**: Single project (desktop application)
**Performance Goals**: 30+ fps screen capture, <5% CPU overhead during recording, <2s startup time, ≥2x real-time export speed
**Constraints**: <200MB baseline memory, <1GB during recording, offline-only (no network), cross-platform parity
**Scale/Scope**: Single-user desktop app, ~5-10 primary UI screens, support for multi-monitor setups

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Code Quality First ✅
- **Status**: PASS
- Rust language enforces memory safety and type correctness
- Plan includes `cargo clippy`, `cargo fmt`, and zero-warning policy
- Error handling via `Result<T, E>` types (idiomatic Rust)
- Dependencies will be documented and justified in Cargo.toml

### II. Testing Rigor ✅
- **Status**: PASS
- Plan includes unit tests (`#[cfg(test)]`), integration tests (`tests/`), and benchmarks (`criterion`)
- Coverage target: 90%+ for critical paths (recording, storage, export)
- Cross-platform testing required (macOS, Linux, Windows)
- Test-first development workflow enforced

### III. User Experience Consistency ✅
- **Status**: PASS
- iced GUI framework provides cross-platform consistency
- Spec requires platform-specific keyboard shortcuts (Cmd/Ctrl)
- Platform-appropriate config storage (XDG/AppData/Application Support)
- Actionable error messages specified in requirements
- <100ms feedback requirement for UI interactions

### IV. Performance by Design ✅
- **Status**: PASS
- Performance goals align with constitution requirements:
  - <5% CPU overhead (constitution requirement)
  - <2s startup time (constitution requirement)
  - ≥2x real-time export speed (constitution requirement)
  - <200MB baseline, <1GB during recording (meets <2GB constitution target)
- Benchmarking required using `criterion`
- Release builds will use LTO and optimized codegen

### Platform-Specific Requirements ✅
- **Status**: PASS
- Targets macOS, Linux, Windows as required
- Plan includes platform-specific implementations using `#[cfg(target_os = "...")]`
- UI framework (iced) is cross-platform compatible

### Security & Privacy Standards ✅
- **Status**: PASS
- No telemetry (offline-only constraint)
- Local storage only (no database, file system based)
- Input validation planned for file operations
- No external network dependencies

**Overall Gate Status**: ✅ **PASSED** - All constitution requirements met. No violations requiring justification.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
src/
├── main.rs              # Application entry point, iced application setup
├── ui/                  # GUI components and state management
│   ├── mod.rs
│   ├── app.rs           # Main application state and message handling
│   ├── screens/         # Different UI screens/views
│   │   ├── home.rs      # Main recording controls screen
│   │   ├── settings.rs  # Settings/preferences screen
│   │   └── preview.rs   # Camera/audio preview components
│   └── widgets/         # Reusable UI components
│       ├── camera_overlay.rs  # Camera overlay widget
│       ├── controls.rs        # Recording controls (start/stop/pause)
│       └── indicators.rs      # Visual indicators (timer, levels)
├── recording/           # Core recording functionality
│   ├── mod.rs
│   ├── screen.rs        # Screen capture (platform-specific traits + impls)
│   ├── camera.rs        # Camera capture
│   ├── audio.rs         # Audio capture
│   ├── encoder.rs       # Video encoding and muxing
│   └── session.rs       # Recording session management
├── platform/            # Platform-specific implementations
│   ├── mod.rs
│   ├── macos.rs         # macOS-specific screen capture, permissions
│   ├── linux.rs         # Linux-specific (X11/Wayland)
│   └── windows.rs       # Windows-specific
├── config/              # Configuration and preferences
│   ├── mod.rs
│   ├── settings.rs      # User settings/preferences
│   └── storage.rs       # Config file I/O (TOML/JSON)
└── error.rs             # Error types using thiserror

tests/
├── integration/         # End-to-end integration tests
│   ├── recording_flow.rs      # Complete recording workflows
│   ├── camera_overlay.rs      # Camera overlay integration
│   └── audio_sync.rs          # Audio/video synchronization
└── unit/                # Unit tests (also in #[cfg(test)] modules)

benches/                 # Performance benchmarks (criterion)
└── recording.rs         # Benchmark recording overhead, encoding speed

Cargo.toml              # Dependencies with justification comments
rustfmt.toml            # Rust formatting configuration
.clippy.toml            # Clippy linting configuration
```

**Structure Decision**: Single project structure chosen. This is a standalone desktop application with no separate backend/frontend or mobile components. All code lives in `src/` with clear separation of concerns:
- `ui/` - iced GUI layer
- `recording/` - Core recording logic (screen, camera, audio, encoding)
- `platform/` - Platform-specific implementations using conditional compilation
- `config/` - Settings persistence

This structure supports cross-platform development with platform-agnostic traits in `recording/` and platform-specific implementations in `platform/`.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No complexity violations. All constitution checks passed.
