# Changelog

All notable changes to Turk-OS are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Turk-OS has no releases until version 1.0, which is tagged `v1.0` at the end of Phase 15. Until then, each finished
phase milestone gets its own heading under *Unreleased*, with the date its exit criteria were met.

Sections used inside each entry: **Added** for new features, **Changed** for changes to existing behaviour,
**Fixed** for bug fixes, **Removed** for removed features, **Docs** for the study plan and documentation.

## [Unreleased]

### Project setup

#### Added
- Git repository with the source tree skeleton from the roadmap: `boot/`, `kernel/` (`arch/i386`, `mm`, `proc`,
  `sync`, `fs`, `drivers`, `syscall`), `lib/`, `include/`, `user/` (`libc`, `init`, `sh`, `bin`), `tests/` and
  `tools/`.
- `.gitignore` for build output, disk images, generated root file system files, debug logs, editor files and the
  local `Books/` folder.
- `.gitattributes` with LF line endings and PDFs marked as binary and generated.
- MIT License.

#### Docs
- Study plan: the roadmap and sixteen phase documents (LaTeX sources in `docs/`, PDFs in `docs/pdf/`).
- `README.md` with the project overview, architecture, roadmap, build and debugging reference, and milestone tracker.
- `docs/README.md` with the study plan index and how to build the PDFs.
- `docs/setup.md` with the development environment setup for Arch Linux, Fedora, Ubuntu/Debian, macOS and Windows
  (WSL2).
- `docs/journal.md`, the lab journal template.

<!--
Template for a phase milestone:

### Phase N: Title (YYYY-MM-DD)

#### Added
- ...

#### Fixed
- ...
-->
