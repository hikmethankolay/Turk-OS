# Development environment setup

This guide takes your machine to a working Turk-OS workshop: host tools, an `i686-elf` cross compiler, QEMU, GDB
and an editor that understands the code. It covers tasks 0.1–0.3 and 0.7 of [Phase 0](pdf/phase-00-foundations.pdf)
for every supported platform, in copy-and-paste form.

Turk-OS builds the same way on every platform below: the same cross compiler, the same QEMU, the same Makefile.
Only package names and a few tool names differ. When a command from a phase document isn't found on your system,
look it up in [section 4](#4-tool-names-on-each-platform). Where this guide and a phase document disagree about
anything else, the phase document wins.

## Contents

1. [Supported platforms](#1-supported-platforms)
2. [Install the host tools](#2-install-the-host-tools):
   [Arch Linux](#arch-linux) · [Fedora](#fedora) · [Ubuntu and Debian](#ubuntu-and-debian) ·
   [Other Linux distributions](#other-linux-distributions) · [macOS](#macos) · [Windows (WSL2)](#windows-wsl2)
3. [Build the i686-elf cross compiler](#3-build-the-i686-elf-cross-compiler)
4. [Tool names on each platform](#4-tool-names-on-each-platform)
5. [Verify the toolchain](#5-verify-the-toolchain)
6. [Smoke-test QEMU and GDB](#6-smoke-test-qemu-and-gdb)
7. [Editor setup](#7-editor-setup)
8. [LaTeX for the study plan (optional)](#8-latex-for-the-study-plan-optional)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Supported platforms

| Platform | Support | How | Cross compiler |
|---|---|---|---|
| **Arch Linux** | Full | Native | Build from source ([section 3](#3-build-the-i686-elf-cross-compiler)), or from the AUR |
| **Fedora** | Full | Native | Build from source |
| **Ubuntu 22.04+, Debian 12+** | Full | Native | Build from source |
| Other Linux | Should work | Install the equivalent packages | Build from source |
| **macOS** (Intel and Apple Silicon) | Works, with two gaps | Homebrew | Prebuilt from Homebrew |
| **Windows 10 / 11** | Full, through WSL2 | Ubuntu inside WSL2 | Build from source inside WSL2 |

On macOS the two gaps are that the GRUB and GDB tools have prefixed names, and that the TurkFS image can't be
loop-mounted from the host (Phase 13). Details are in the [macOS](#macos) section.

Every platform ends up with the same set of tools:

| Tool | Used for | From |
|---|---|:---:|
| `i686-elf-gcc`, `i686-elf-ld`, `i686-elf-objdump`, `i686-elf-readelf`, `i686-elf-nm` | Compiling, linking and inspecting the kernel and user programs | Phase 0 |
| `nasm` | Boot sector, loader, interrupt stubs, context switch | Phase 1 |
| `qemu-system-i386` | Running, testing and debugging the OS | Phase 0 |
| `gdb` | Source-level debugging over QEMU's GDB stub | Phase 1 |
| `grub-mkrescue` / `grub2-mkrescue` with `xorriso` and `mtools` | Building the bootable ISO | Phase 1 |
| `grub-file` / `grub2-file` | Checking that the kernel has a valid Multiboot header | Phase 1 |
| Host `gcc`, `make`, `git` | Host unit tests, builds, version control | Phase 0 |
| `qemu-img` | Creating disk images | Phase 12 |
| `mkfs.minix`, `fsck.minix` | Creating and checking TurkFS images | Phase 13 |
| `latexmk` and TeX Live | Rebuilding the study plan PDFs (optional) | any time |

> Installing GRUB packages only installs the tools. It doesn't touch your computer's own boot loader; that only
> happens when someone runs `grub-install`, which this project never does.

---

## 2. Install the host tools

Follow the subsection for your platform, then continue with [section 3](#3-build-the-i686-elf-cross-compiler)
(except on macOS, and on Arch if you use the AUR). On any Linux distribution, if your editor is VS Code installed as
a **Flatpak**, also do the [Flatpak step](#vs-code-installed-as-a-flatpak-linux) in section 7.

### Arch Linux

```sh
sudo pacman -S --needed base-devel git nasm qemu-desktop gdb libisoburn mtools grub
```

| Package | Why |
|---|---|
| `base-devel` | `gcc`, `make`, `bison`, `flex`, `texinfo`; GCC's own dependencies (`gmp`, `mpfr`, `libmpc`) come with it |
| `qemu-desktop` | `qemu-system-i386`, `qemu-img` and the GTK/SDL display modules. Plain `qemu-system-x86` runs, but opens no window |
| `libisoburn` | Arch's package for `xorriso` |
| `grub` | `grub-mkrescue`, `grub-file` and the `i386-pc` modules |

`mkfs.minix` and `fsck.minix` come with `util-linux`, which every Arch system already has.

**Cross compiler from the AUR (optional).** Instead of section 3 you can install the cross toolchain with an AUR
helper. It still compiles from source, but pacman then tracks and updates it, and it installs into `/usr`, so no
`PATH` change is needed:

```sh
yay -S i686-elf-binutils i686-elf-gcc      # or paru, or makepkg by hand
```

### Fedora

```sh
sudo dnf install gcc make git nasm qemu-system-x86 qemu-img gdb xorriso mtools \
                 grub2-tools grub2-tools-extra grub2-pc-modules util-linux
# What GCC needs to build from source
sudo dnf install bison flex gmp-devel mpfr-devel libmpc-devel texinfo
```

| Package | Why |
|---|---|
| `qemu-img` | `qemu-img`, which Fedora packages separately from `qemu-system-x86` |
| `grub2-tools` | `grub2-file` (Fedora's GRUB tools carry a `2` in their names) |
| `grub2-tools-extra` | `grub2-mkrescue` |
| `grub2-pc-modules` | The BIOS (`i386-pc`) GRUB modules the ISO needs |

### Ubuntu and Debian

```sh
sudo apt update
sudo apt install build-essential git curl nasm qemu-system-x86 qemu-system-gui qemu-utils gdb \
                 xorriso mtools grub-pc-bin grub-common util-linux
# What GCC needs to build from source
sudo apt install bison flex libgmp-dev libmpfr-dev libmpc-dev texinfo
```

| Package | Why |
|---|---|
| `build-essential` | Host `gcc`, `make` and the C library headers |
| `qemu-system-x86` | `qemu-system-i386` |
| `qemu-system-gui` | The QEMU window (GTK); without it QEMU can only run headless |
| `qemu-utils` | `qemu-img`, which Debian packages separately |
| `grub-common` | `grub-mkrescue` and `grub-file` (named without the `2`) |
| `grub-pc-bin` | The BIOS (`i386-pc`) GRUB modules. Without them `grub-mkrescue` still builds an ISO, but it only boots on UEFI, and QEMU's BIOS says *No bootable device* |

### Other Linux distributions

Install the equivalents of the Debian list: GCC, Make and Git; NASM; QEMU with `qemu-system-i386`, `qemu-img` and a
display module; GDB; GRUB 2 with its `i386-pc` modules, plus `xorriso` and `mtools`; `util-linux` (for
`mkfs.minix`); and, to build the cross compiler, Bison, Flex, Texinfo and the GMP, MPFR and MPC development packages.
Then continue with section 3. The OSDev Wiki page
[GCC Cross-Compiler](https://wiki.osdev.org/GCC_Cross-Compiler#Installing_Dependencies) lists the package names for
more distributions.

### macOS

Homebrew has every tool, including a prebuilt cross compiler, so macOS skips section 3. Install
[Homebrew](https://brew.sh) first if you don't have it, then:

```sh
brew install i686-elf-binutils i686-elf-gcc i686-elf-grub i386-elf-gdb \
             nasm qemu xorriso mtools util-linux make coreutils
```

The Homebrew `util-linux` is *keg-only*: its programs are not on your `PATH`. Add the folder with `mkfs.minix` and
`fsck.minix` to `~/.zshrc`:

```sh
echo 'export PATH="$(brew --prefix util-linux)/sbin:$PATH"' >> ~/.zshrc
```

Things that work differently on macOS:

| Topic | On macOS |
|---|---|
| Cross compiler | `i686-elf-gcc`, `i686-elf-ld` and friends are already on your `PATH` from Homebrew |
| GRUB tools | `i686-elf-grub-mkrescue` and `i686-elf-grub-file` instead of `grub2-mkrescue` / `grub2-file` |
| Debugger | `i386-elf-gdb` instead of `gdb` (works on Intel and Apple Silicon) |
| `make` | macOS ships GNU Make 3.81 from 2006. Use `gmake` (Homebrew's current GNU Make) for this project |
| `timeout` (`make test`, Phase 7) | `gtimeout`, from `coreutils` |
| Host compiler | `gcc` is Apple clang. The Phase 0 host tests with `-fsanitize=address,undefined` work with it |
| Loop mounts (Phase 13) | macOS can't mount a MINIX image, so the optional `sudo mount -o loop` step doesn't work. Put files onto TurkFS from Turk-OS itself, with the planned `tools/mkturkfs.c`, or in a Linux virtual machine |
| QEMU monitor | Start QEMU with `-monitor stdio`, or use the *View* menu in the QEMU window |
| CPU count | `nproc` doesn't exist; use `sysctl -n hw.ncpu` if a command asks for it |

[Section 4](#4-tool-names-on-each-platform) shows how to make the Makefile pick up the prefixed names by itself.

### Windows (WSL2)

Turk-OS is built inside **WSL2** (Windows Subsystem for Linux) running Ubuntu. Native Windows tool ports such as MSYS2
or Cygwin aren't supported: they lack a working `grub-mkrescue`, `mkfs.minix` and loop mounts, which the plan needs.

1. **Install WSL2 and Ubuntu.** Open PowerShell *as Administrator*:

   ```powershell
   wsl --install -d Ubuntu
   ```

   Restart when asked, open **Ubuntu** from the Start menu, and create your Linux user name and password.

2. **Update WSL.** This brings in WSLg, which shows Linux windows, such as QEMU's, on the Windows desktop:

   ```powershell
   wsl --update
   ```

3. **Install the tools.** In the Ubuntu terminal, follow [Ubuntu and Debian](#ubuntu-and-debian), then build the
   cross compiler ([section 3](#3-build-the-i686-elf-cross-compiler)). Every later command in this guide and in the
   phase documents runs in this Ubuntu terminal.

4. **Clone the repository inside Linux**, not under `/mnt/c`:

   ```sh
   cd ~
   git clone https://github.com/hikmethankolay/Turk-OS.git
   ```

Things that work differently on WSL2:

| Topic | What to do |
|---|---|
| Where the repository lives | In the Linux home folder (`~/Turk-OS`). Files under `/mnt/c/...` are much slower to build from and bring Windows permissions and line endings along. To open the folder in Windows Explorer, run `explorer.exe .` inside it. |
| Editor | Install VS Code on Windows with the **WSL** extension, then run `code .` in the repository from the Ubuntu terminal. The compiler, terminal and language server all run inside Linux. See [section 7](#7-editor-setup). |
| QEMU window | Appears through WSLg. If no window opens, run QEMU with `-display curses` to draw the VGA text screen in the terminal (press Esc then 2 for the monitor, where `quit` stops QEMU), or with `-display none -serial stdio` to use only the serial log (Ctrl+C stops QEMU). |
| Git | Use `git` inside WSL. Don't work on the same clone from Git for Windows as well. |
| Writing the ISO to a USB stick (Phase 15) | WSL can't see USB disks. Copy the ISO to Windows (`cp build/turkos.iso /mnt/c/Users/<you>/Downloads/`) and write it with [Rufus](https://rufus.ie) in *DD image* mode. |

---

## 3. Build the i686-elf cross compiler

**For:** every Linux distribution and WSL2. Arch users can take the AUR packages instead. **Skip on macOS.**

Your system's `gcc` builds programs for Linux (or macOS). It assumes things that are false inside a kernel: that a C
library exists, that the stack protector runtime is available, that code should be position-independent. A **cross
compiler** that targets `i686-elf` assumes nothing, which prevents a whole family of confusing bugs. Building it
takes 20–40 minutes, once. This follows the OSDev Wiki page
[GCC Cross-Compiler](https://wiki.osdev.org/GCC_Cross-Compiler).

### 3.1 Set the versions, target and install prefix

The versions below were the newest releases in September 2026. Check [ftp.gnu.org/gnu](https://ftp.gnu.org/gnu/)
(`binutils/` and `gcc/`) and use newer ones if they exist.

```sh
export BINUTILS_VERSION=2.47
export GCC_VERSION=16.2.0
export PREFIX="$HOME/opt/cross"
export TARGET=i686-elf
export PATH="$PREFIX/bin:$PATH"
```

The tools install into your home folder, so no `sudo` is needed and your system compiler is never touched. Run the
rest of this section in the **same terminal**, because the variables above only live there.

### 3.2 Get the sources

```sh
mkdir -p ~/src && cd ~/src
curl -LO https://ftp.gnu.org/gnu/binutils/binutils-$BINUTILS_VERSION.tar.xz
curl -LO https://ftp.gnu.org/gnu/gcc/gcc-$GCC_VERSION/gcc-$GCC_VERSION.tar.xz
tar xf binutils-$BINUTILS_VERSION.tar.xz
tar xf gcc-$GCC_VERSION.tar.xz
```

### 3.3 Build binutils

Always build in a separate folder, never inside the source tree.

```sh
mkdir -p ~/src/build-binutils && cd ~/src/build-binutils
../binutils-$BINUTILS_VERSION/configure --target=$TARGET --prefix="$PREFIX" \
    --with-sysroot --disable-nls --disable-werror
make -j"$(nproc)" && make install
```

### 3.4 Build GCC

GCC's `configure` needs the binutils you just installed on the `PATH` (step 3.1 took care of that).

```sh
mkdir -p ~/src/build-gcc && cd ~/src/build-gcc
../gcc-$GCC_VERSION/configure --target=$TARGET --prefix="$PREFIX" \
    --disable-nls --enable-languages=c --without-headers
make -j"$(nproc)" all-gcc all-target-libgcc
make install-gcc install-target-libgcc
```

`libgcc` matters: GCC sometimes emits calls into it, for example for 64-bit division, and the kernel links with
`-lgcc` from Phase 2 on.

### 3.5 Make it permanent

```sh
echo 'export PATH="$HOME/opt/cross/bin:$PATH"' >> ~/.bashrc     # ~/.zshrc if your shell is zsh
```

Open a new terminal so the change takes effect. The folders in `~/src` can be deleted afterwards.

### If the cross build fails

You can start Phase 1 with the host compiler while you fix it, as *The little book about OS development* does. Use
these flags and link with `ld -m elf_i386`:

```text
-m32 -ffreestanding -fno-pie -fno-stack-protector -fno-builtin -nostdlib -fcf-protection=none
```

On Ubuntu and Debian, `-m32` also needs `sudo apt install gcc-multilib`. Switch to the cross compiler **before
Phase 3**: the host compiler's hidden assumptions cause bugs that are very hard to recognise.

---

## 4. Tool names on each platform

A few tools have different names on different platforms. The phase documents pick one name in their examples, so
when a command isn't found on your system, look it up here:

| Purpose | Arch | Fedora | Ubuntu / Debian / WSL2 | macOS |
|---|---|---|---|---|
| Install packages | `pacman` | `dnf` | `apt` | `brew` |
| Build the ISO | `grub-mkrescue` | `grub2-mkrescue` | `grub-mkrescue` | `i686-elf-grub-mkrescue` |
| Check the Multiboot header | `grub-file` | `grub2-file` | `grub-file` | `i686-elf-grub-file` |
| Debugger | `gdb` | `gdb` | `gdb` | `i386-elf-gdb` |
| GNU Make | `make` | `make` | `make` | `gmake` |
| Time limit for `make test` | `timeout` | `timeout` | `timeout` | `gtimeout` |
| Cross compiler location | `~/opt/cross/bin`, or `/usr/bin` (AUR) | `~/opt/cross/bin` | `~/opt/cross/bin` | Homebrew |

So that one Makefile works everywhere, let it find the tools instead of hard-coding them. When you write the Makefile
in Phase 1, put this near the top and use `$(GRUB_MKRESCUE)`, `$(GDB)` and `$(TIMEOUT)` in the recipes:

```make
# Tool names differ between platforms (see docs/setup.md, section 4).
# The first name found on the PATH wins; override with e.g. `make GDB=/path/to/gdb`.
find_tool     = $(firstword $(foreach t,$(1),$(shell command -v $(t) 2>/dev/null)))
GRUB_MKRESCUE := $(call find_tool,grub2-mkrescue grub-mkrescue i686-elf-grub-mkrescue)
GRUB_FILE     := $(call find_tool,grub2-file grub-file i686-elf-grub-file)
GDB           := $(call find_tool,i386-elf-gdb gdb)
TIMEOUT       := $(call find_tool,timeout gtimeout)
```

---

## 5. Verify the toolchain

Open a **new** terminal (the one your editor uses) and run each command. On macOS use the names from section 4.

| Command | Expected result |
|---|---|
| `i686-elf-gcc --version` | `i686-elf-gcc (GCC) 16.2.0` (or the version you built) |
| `i686-elf-ld --version` | `GNU ld (GNU Binutils) 2.47` |
| `nasm -v` | `NASM version ...` |
| `qemu-system-i386 --version` | `QEMU emulator version ...` |
| `gdb --version` | `GNU gdb ...` |
| `grub-mkrescue --version` (or your platform's name from section 4) | `... (GRUB) 2.x` |
| `command -v xorriso mkfs.minix` | Two paths |

Then compile something freestanding and look at it. This is also Phase 0's first assembly-reading exercise:

```sh
mkdir -p /tmp/turkos-check && cd /tmp/turkos-check
printf 'int add(int a, int b) { return a + b; }\n' > add.c
i686-elf-gcc -std=gnu11 -ffreestanding -O0 -S -masm=intel add.c -o add.s   # read add.s
i686-elf-gcc -std=gnu11 -ffreestanding -O0 -c add.c -o add.o
i686-elf-objdump -d -M intel add.o     # machine code next to the instructions
i686-elf-readelf -S add.o              # sections: .text .data .bss ...
i686-elf-nm add.o                      # symbols the linker will see
```

`i686-elf-readelf -h add.o` should report `Class: ELF32` and `Machine: Intel 80386`.

---

## 6. Smoke-test QEMU and GDB

Start the emulated PC with nothing to boot:

```sh
qemu-system-i386 -monitor stdio
```

SeaBIOS tries every device and reports that there is no bootable disk. That screen is exactly where your own code
takes over in Phase 1. At the `(qemu)` prompt in the terminal, run `info registers` and find `EIP` and `CR0`. Type
`quit` to leave.

In the QEMU window on Linux and WSL2, `Ctrl+Alt+2` switches to the monitor, `Ctrl+Alt+1` back to the screen, and
`Ctrl+Alt+G` releases the mouse.

GDB attaches to QEMU's built-in stub: `-s` listens on TCP port 1234 and `-S` freezes the CPU until GDB says continue.
In one terminal:

```sh
qemu-system-i386 -s -S
```

In a second terminal (`i386-elf-gdb` on macOS):

```sh
gdb -ex "target remote localhost:1234"
(gdb) info registers
(gdb) stepi
(gdb) continue
```

From Phase 1 on, `make debug` does this for you with the kernel's symbols loaded.

---

## 7. Editor setup

Any editor works. These notes cover VS Code, which the plan assumes, and the language server.

### VS Code installed as a Flatpak (Linux)

The Flatpak build of VS Code runs its integrated terminal inside a sandbox, where host programs like `nasm` and
`qemu-system-i386` are invisible. This applies to any distribution. Either work in a normal terminal window, or add a
terminal profile that starts a shell on the host. Open *Preferences: Open User Settings (JSON)* and add:

```json
"terminal.integrated.profiles.linux": {
  "host-bash": { "path": "/usr/bin/flatpak-spawn", "args": ["--host", "--env=TERM=xterm-256color", "bash"] }
},
"terminal.integrated.defaultProfile.linux": "host-bash"
```

**Check:** in a new VS Code terminal, `which nasm` prints `/usr/bin/nasm`. If VS Code came from a `.rpm`, `.deb`, the
AUR or Snap, skip this step.

### VS Code on Windows with WSL2

Install VS Code on Windows (not inside Ubuntu), add the **WSL** extension, then in the Ubuntu terminal run
`code .` inside the repository. The window title shows `[WSL: Ubuntu]`; its terminal, extensions and language server
run inside Linux and see the cross compiler.

### VS Code on macOS

Install it normally. Its terminal sees everything Homebrew installed.

### Code intelligence with clangd

The editor needs to know that the code is compiled with `i686-elf-gcc` and `-Iinclude`, otherwise it flags every
kernel header as missing. **clangd** (or the Microsoft C/C++ extension) reads a `compile_commands.json` file, which
[Bear](https://github.com/rizsotto/Bear) generates from a real build:

| Platform | Install Bear |
|---|---|
| Arch | `sudo pacman -S bear` |
| Fedora | `sudo dnf install bear` |
| Ubuntu / Debian / WSL2 | `sudo apt install bear` |
| macOS | `brew install bear` |

Then run `bear -- make` (or `bear -- gmake` on macOS) once the Makefile exists. `compile_commands.json` and clangd's
`.cache/` are already in `.gitignore`.

### Two settings that matter everywhere

- **Tabs in Makefiles:** recipe lines must start with a real Tab character. Make sure your editor doesn't turn tabs
  into spaces in files named `Makefile`.
- **Line endings:** the repository's `.gitattributes` keeps every text file in LF. Don't change that in the editor,
  especially on Windows: shell scripts and Makefiles break with CRLF endings.

---

## 8. LaTeX for the study plan (optional)

You only need this to rebuild the PDFs in `docs/pdf/` after changing a `.tex` file. The finished PDFs are already in
the repository. The style needs `latexmk`, pdfLaTeX and a set of packages
([full list](README.md#building-the-pdfs)).

| Platform | Install |
|---|---|
| Arch | `sudo pacman -S --needed texlive-basic texlive-latex texlive-latexrecommended texlive-latexextra texlive-fontsrecommended texlive-fontsextra texlive-pictures texlive-binextra` |
| Fedora | `sudo dnf install latexmk texlive-scheme-full`, or a smaller scheme plus packages by file name (see [docs/README.md](README.md#requirements)) |
| Ubuntu / Debian / WSL2 | `sudo apt install latexmk texlive-latex-extra texlive-fonts-extra texlive-fonts-recommended texlive-pictures` |
| macOS | `brew install --cask mactex-no-gui` (includes `latexmk`), then open a new terminal |

Then build with `make -C docs` from the repository root.

---

## 9. Troubleshooting

| Platform | Symptom | Likely cause | Fix |
|---|---|---|---|
| All | GCC build stops with missing `gmp.h` or `mpfr.h` | Build dependencies missing | Install the GMP, MPFR and MPC development packages for your platform (section 2), then rerun `configure` |
| All | `configure: error: building in source directory` | Built inside the source tree | Use the separate `build-binutils` / `build-gcc` folders |
| All | GCC `configure` can't find `i686-elf-as` | binutils not installed, or not on `PATH` | Finish step 3.3, and repeat the exports from step 3.1 in this terminal |
| All | `i686-elf-gcc` found in one terminal but not another | `PATH` set only in that shell | Put the export in `~/.bashrc` (or `~/.zshrc`) and open a new terminal |
| All | A command from a phase document isn't found, for example `grub2-mkrescue: command not found` | The tool has a different name on your platform | Look it up in section 4, or use the Makefile variables from section 4 |
| All | `grub-mkrescue` fails with a `xorriso` or `mformat` error | Missing ISO tools | Install `xorriso` (`libisoburn` on Arch) and `mtools` |
| All | QEMU window opens but nothing is printed in the terminal | Serial output not redirected | Add `-serial stdio` (from Phase 2 on) |
| Arch | QEMU runs but no window opens | Only `qemu-system-x86` installed, without a display module | `sudo pacman -S qemu-desktop` |
| Linux (Flatpak VS Code) | `nasm: command not found` in VS Code, but it works in a normal terminal | Flatpak sandbox | Use the host terminal profile from [section 7](#vs-code-installed-as-a-flatpak-linux) |
| macOS | Strange errors from `make` | macOS's GNU Make 3.81 | Use `gmake` |
| macOS | `mkfs.minix: command not found` | `util-linux` is keg-only | Add `$(brew --prefix util-linux)/sbin` to `PATH` (see [macOS](#macos)) |
| macOS | `timeout: command not found` in `make test` | GNU `timeout` isn't part of macOS | `brew install coreutils` and use `gtimeout` (section 4) |
| Ubuntu / Debian / WSL2 | The ISO builds, but QEMU says *No bootable device* | `grub-pc-bin` missing, so the ISO has no BIOS boot code | `sudo apt install grub-pc-bin`, then rebuild the ISO |
| Ubuntu / Debian / WSL2 | `qemu-img: command not found` | Packaged separately | `sudo apt install qemu-utils` |
| WSL2 | QEMU prints `gtk initialization failed` or no window appears | WSLg missing or outdated | Run `wsl --update` in PowerShell and restart WSL (`wsl --shutdown`); meanwhile use `-display curses` or `-display none -serial stdio` |
| WSL2 | Builds are very slow | Repository under `/mnt/c` | Clone into the Linux home folder (`~/Turk-OS`) |
| WSL2 | `$'\r': command not found` or `/bin/sh^M: bad interpreter` | Files checked out with Windows (CRLF) line endings | Clone inside WSL, not with Git for Windows; in an existing clone run `git add --renormalize .` |

When a problem takes more than an hour, write down the hypothesis in the [lab journal](journal.md) before each
attempt. The OSDev Wiki's [GCC Cross-Compiler](https://wiki.osdev.org/GCC_Cross-Compiler) page lists more known
problems.
