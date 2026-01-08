<!--
Sync Impact Report
==================
Version: 1.0.0 (initial constitution)
Ratification Date: 2026-01-02
Last Amended: 2026-01-02

Principles Defined:
- Code Quality First
- Testing Rigor
- User Experience Consistency
- Performance by Design

Templates Status:
✅ plan-template.md - Reviewed, constitution check section aligns
✅ spec-template.md - Reviewed, requirements structure supports principles
✅ tasks-template.md - Reviewed, test-first workflow supported
✅ agent-file-template.md - Reviewed, no conflicts
✅ checklist-template.md - Reviewed, no conflicts

Follow-up TODOs:
- None (all placeholders filled)
-->

# Soufle Constitution

## Core Principles

### I. Code Quality First

**All Rust code written for Soufle MUST adhere to the following non-negotiable standards:**

- **Compiler Warnings**: Code MUST compile with `#![deny(warnings)]` or `cargo clippy -- -D warnings` with zero warnings before commit
- **Idiomatic Rust**: Follow Rust API guidelines; prefer `Result<T, E>` over panics, use `Option<T>` explicitly, avoid `.unwrap()` in production code paths
- **Self-Documenting**: Code readability takes precedence over cleverness; complex logic MUST include explanatory comments
- **Single Responsibility**: Functions and modules MUST have one clear purpose
- **No Dead Code**: Unused code, commented-out code, and deprecated functions MUST be removed, not left in place
- **Dependency Hygiene**: Every crate dependency MUST be justified in Cargo.toml comments; avoid adding crates for trivial functionality
- **Error Handling**: All errors MUST be properly typed using `thiserror` or similar; error messages MUST be actionable

**Rationale**: Soufle is a privacy-focused offline tool. Users trust us with their recordings. Rust's safety guarantees combined with strict quality standards reduce bugs and security vulnerabilities, ensuring user data safety and product reliability.

### II. Testing Rigor (NON-NEGOTIABLE)

**Testing is mandatory and MUST follow these rules:**

- **Test-First Development**: For new features, tests MUST be written before implementation where practical
- **Red-Green-Refactor**: Tests MUST fail initially (`cargo test` shows failures), then pass after implementation
- **Coverage Targets**:
  - Critical paths (recording, storage, export) MUST have 90%+ code coverage (use `cargo-tarpaulin` or `cargo-llvm-cov`)
  - All user-facing features MUST have integration tests
  - Edge cases and error scenarios MUST have dedicated tests
- **Test Types Required**:
  - **Unit Tests**: All business logic and utilities (`#[cfg(test)]` modules)
  - **Integration Tests**: User journeys (start recording → save → export) in `tests/` directory
  - **Performance Tests**: Benchmarks using `criterion` for recording overhead, export speed, storage efficiency
- **Platform Testing**: Tests MUST pass on macOS, Linux, and Windows before merge

**Rationale**: Offline recording tools have no server-side fallback. Bugs directly impact user data. Comprehensive testing prevents data loss and ensures consistent behavior across platforms.

### III. User Experience Consistency

**Every user-facing feature MUST maintain these standards across all platforms (macOS, Linux, Windows):**

- **Instant Feedback**: All user actions MUST provide immediate visual or auditory feedback (<100ms)
- **Error Messaging**: Errors MUST be actionable (e.g., "Microphone access denied. Enable in System Preferences → Privacy → Microphone" on macOS, equivalent guidance for Linux/Windows)
- **Keyboard Shortcuts**: All primary actions MUST have keyboard shortcuts following platform conventions (Cmd on macOS, Ctrl on Windows/Linux)
- **Native Look & Feel**: UI MUST follow platform design guidelines (using native toolkit or consistent theming)
- **Accessibility**: MUST support screen readers, keyboard navigation, and high-contrast modes on all platforms
- **State Persistence**: User preferences and session state MUST be saved automatically using platform-appropriate storage (XDG on Linux, AppData on Windows, Application Support on macOS)
- **Cross-Platform Consistency**: Core workflows MUST behave identically across platforms while respecting platform UX conventions

**Rationale**: Users switch from paid tools (like Loom) to Soufle for simplicity and control. Inconsistent UX across platforms creates friction and erodes trust. Cross-platform desktop apps must feel native, not bolted-on.

### IV. Performance by Design

**Performance is not an optimization phase—it is a design requirement. Rust enables zero-cost abstractions—use them:**

- **Recording Overhead**: Screen capture MUST NOT cause visible frame drops or lag (<5% CPU overhead target)
- **Startup Time**: Application MUST launch in <2 seconds on target hardware (cold start on modern mid-range systems)
- **Export Speed**: Video export MUST process at ≥2x real-time speed (5-minute video exports in <2.5 minutes)
- **Storage Efficiency**: Recordings MUST use efficient codecs (H.264/H.265/VP9) with user-configurable quality
- **Memory Footprint**: Application MUST run comfortably on 8GB RAM systems (<200MB baseline, <1GB during active recording)
- **Profiling Required**: Any performance-critical code MUST be profiled using `cargo flamegraph`, `perf`, or platform profilers before and after changes
- **Compilation**: Release builds MUST use `lto = "fat"` and `codegen-units = 1` for maximum optimization
- **Async Runtime**: Use `tokio` or `async-std` efficiently; avoid blocking operations on async threads

**Rationale**: Soufle competes with cloud-based tools that offload processing to servers. To succeed as an offline tool, performance must be exceptional to justify local resource usage. Rust's zero-cost abstractions enable this—use them.

## Code Review & Quality Gates

**All code changes MUST pass the following gates before merge:**

### Pre-Commit Checks
- `cargo fmt` applied (rustfmt.toml configuration)
- `cargo clippy -- -D warnings` passes with zero warnings
- `cargo check` and `cargo build --release` succeed
- No `dbg!()`, `println!()`, or `eprintln!()` debug statements in production code (use `log` or `tracing` crates)

### Pre-Merge Review
- At least one approved review from a team member
- All tests pass on all platforms: `cargo test --all-features`
- Code coverage does not decrease (measured by `cargo-tarpaulin` or equivalent)
- No new crate dependencies without justification in PR description and Cargo.toml comments
- `cargo audit` passes (no known vulnerabilities in dependencies)

### Performance Gate
- If code touches recording, export, or rendering:
  - Benchmark results included in PR (using `criterion` or similar)
  - No regressions >5% without explicit justification
  - Memory usage profiled (using `valgrind` on Linux or platform equivalents)

### Documentation Gate
- Public APIs MUST have rustdoc comments (`///` for items, `//!` for modules)
- Breaking changes MUST update relevant documentation and increment crate version appropriately
- New features MUST update user guides or quickstart
- Examples MUST compile and run (`cargo test --doc`)

## Security & Privacy Standards

**Soufle is an offline-first, privacy-focused tool. The following MUST be enforced:**

- **No Telemetry by Default**: User behavior MUST NOT be tracked without explicit opt-in
- **Local Data Only**: Recordings MUST be stored locally unless user explicitly enables cloud sync (future feature)
- **Secure Storage**: Sensitive settings (API keys, if any) MUST use platform-native secure storage (Keychain on macOS, Secret Service on Linux, Credential Manager on Windows)
- **Dependency Audits**: Run `cargo audit` before each release; dependencies MUST be audited quarterly for known vulnerabilities
- **Data Deletion**: Users MUST be able to permanently delete all recordings and metadata through the UI
- **Memory Safety**: Leverage Rust's ownership system; `unsafe` blocks MUST be minimized, documented, and reviewed carefully
- **Input Validation**: All user inputs and file operations MUST be validated; use `Path::canonicalize()` to prevent directory traversal

## Platform-Specific Requirements

**Soufle targets macOS, Linux, and Windows. Platform-specific code MUST follow these standards:**

### Conditional Compilation
- Use `#[cfg(target_os = "...")]` for platform-specific implementations
- Provide platform-agnostic traits/interfaces; implement per-platform backends
- Document platform differences in rustdoc comments

### Platform Testing
- CI/CD MUST run tests on all three platforms before merge
- Platform-specific features (e.g., macOS screen recording permissions) MUST have integration tests on that platform

### UI Framework
- Choose a cross-platform Rust GUI framework (e.g., `iced`, `egui`, `slint`, `tauri`) or platform-native approaches
- Justify framework choice in architecture documentation
- UI code MUST handle platform-specific behaviors gracefully (window management, notifications, system tray)

## Governance

**This constitution supersedes all other development practices and standards.**

### Amendment Procedure
- Amendments require documentation in PR description explaining:
  - What is changing and why
  - Impact on existing features
  - Migration plan (if applicable)
- Amendments require approval from at least two project maintainers
- Version bump follows semantic versioning (see below)

### Versioning Policy
- **MAJOR (X.0.0)**: Backward-incompatible principle changes (e.g., removing a core principle)
- **MINOR (1.X.0)**: New principles added or material expansions to existing principles
- **PATCH (1.0.X)**: Clarifications, typo fixes, non-semantic refinements

### Compliance Review
- All PRs MUST verify compliance with constitution principles
- Complexity that violates principles MUST be explicitly justified in plan.md (see Complexity Tracking section)
- Code reviews MUST check for principle violations and flag them before merge

### Runtime Guidance
During active development, refer to this constitution when making architectural decisions, reviewing code, or planning features. If uncertainty arises, default to the most conservative interpretation that upholds code quality, testing rigor, UX consistency, and performance standards.

**Version**: 1.0.0 | **Ratified**: 2026-01-02 | **Last Amended**: 2026-01-02
