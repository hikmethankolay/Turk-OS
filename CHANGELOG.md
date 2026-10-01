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

### Study plan: Stage V, the RISC-V and ARM ports

#### Added
- Skeleton folders for the Stage V layout: `kernel/arch/riscv32/`, `kernel/arch/aarch64/`, `user/arch/i386/`,
  `user/arch/riscv32/`, `user/arch/aarch64/` and `tests/data/`.

#### Docs
- Three new phase documents after Turk-OS 1.0: Phase 16 *Portability: an Architecture Layer*, Phase 17 *The RISC-V
  Port* (RV32 with Sv32 on QEMU `virt` and OpenSBI, optionally the Timur-RV32IMC core) and Phase 18 *The ARM Port*
  (AArch64 at EL1 on QEMU `virt`, optionally a Raspberry Pi 4), ending at Turk-OS 2.0. Stage V has its own
  progress scale; the 1.0 scale of Phases 0–15 is unchanged.
- Roadmap: nineteen phases in five stages, new technical decisions and a decision box on porting after 1.0, the
  Stage V row in the phase map, a 58-week timeline, a figure of Turk-OS 2.0 on three architectures, Stage V risks
  and tracker entries.
- Phase 15: Track E now points to Stage V, and the phase leads into Phase 16. Phases 2, 3, 6, 8 and 10 each gained
  a short portability note.
- `docs/setup.md`: section 9, the toolchains for the ports (cross compilers, QEMU boards, OpenSBI, `dtc`, GDB) on
  every platform; Troubleshooting is now section 10.
- `docs/common/turkos.sty`: GNU as listing styles for RISC-V and AArch64, chips and a legend for the Stage V
  references, the Stage V colour, and a configurable progress-bar label.
- `README.md` and `docs/README.md` describe Stage V and Turk-OS 2.0.

<!--
Template for a phase milestone:

### Phase N: Title (YYYY-MM-DD)

#### Added
- ...

#### Fixed
- ...
-->
