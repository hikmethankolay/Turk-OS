# Turk-OS

**A small UNIX-like operating system for 32-bit x86 PCs, written from scratch in C and NASM.**

Turk-OS boots with GRUB, runs preemptive multitasking with separate address spaces, loads ELF programs from its own
on-disk file system, and drops you into a shell with pipes and redirection. It is built one phase at a time, following
a 16-phase study plan that lives in this repository. Everything runs in QEMU first; real hardware is optional.

<!-- Keep this block in sync with the milestone tracker at the bottom of the file. -->
| | |
|---|---|
| **Current phase** | Phase 0: Foundations and toolchain |
| **Overall progress** | 0 % of the way to Turk-OS 1.0 |
| **Next milestone** | *Workshop ready*: cross toolchain works, skeleton committed, warm-up library passes its tests |
| **Target** | IA-32 (i386), 32-bit protected mode, legacy BIOS boot |
| **Languages** | C (gnu11) and NASM (Intel syntax) |
| **Latest release** | none yet (`v1.0` is the goal of Phase 15) |
| **License** | [MIT](LICENSE) |

---

## Contents

1. [About the project](#about-the-project)
2. [What Turk-OS 1.0 will do](#what-turk-os-10-will-do)
3. [Technical decisions](#technical-decisions)
4. [Architecture](#architecture)
   - [Layers](#layers)
   - [Boot sequence](#boot-sequence)
   - [Virtual memory layout](#virtual-memory-layout)
   - [System call interface](#system-call-interface)
   - [TurkFS on-disk format](#turkfs-on-disk-format)
5. [Roadmap](#roadmap)
6. [Repository layout](#repository-layout)
7. [Getting started](#getting-started)
   - [Host requirements](#host-requirements)
   - [Build and run](#build-and-run)
   - [Running QEMU by hand](#running-qemu-by-hand)
   - [Debugging](#debugging)
   - [Testing](#testing)
8. [The study plan](#the-study-plan)
9. [Books and references](#books-and-references)
10. [Conventions](#conventions)
11. [Milestone tracker](#milestone-tracker)
12. [License](#license)

---

## About the project

Turk-OS is a learning project. It starts from an empty folder, basic C and no assembly experience, and ends at
**Turk-OS 1.0**: a small but complete monolithic kernel with a user-space C library, a shell and a set of core
utilities. Every line of kernel code is written by hand, in the order the study plan gives, and each phase ends with
a concrete milestone that can be tested.

The plan is built around five operating-system textbooks (see [Books and references](#books-and-references)). The
practical steps follow *The little book about OS development*, the theory comes from *Operating Systems: Three Easy
Pieces* and its companions, and every subsystem is compared with the real MINIX 3 source code printed in
*Operating Systems: Design and Implementation*.

**Time budget.** About 43 weeks (roughly ten months) at 10–12 hours a week. Paging (Phase 6), processes (Phase 8)
and file systems (Phase 13) are expected to run over.

**In scope for 1.0:** one CPU, 32-bit x86, BIOS boot through GRUB, text-mode console, IDE disk, a MINIX-compatible
file system, a POSIX-flavoured system call set, a shell and core utilities.

**Out of scope for 1.0** (these are optional tracks after the release, see [Phase 15](docs/pdf/phase-15-hardening-release.pdf)):
graphics, multiprocessor support, networking, a 64-bit port, a RISC-V port, a microkernel experiment and virtualisation.

---

## What Turk-OS 1.0 will do

The table lists what the finished system contains and which phase builds each part.

| Area | Features | Phase |
|---|---|---|
| **Boot** | Multiboot 1 kernel loaded by GRUB 2 (bootable ISO from `grub-mkrescue`) or by `qemu -kernel`; kernel linked at `0xC0100000` in the higher half | 1, 6 |
| **Console and diagnostics** | VGA text driver (colours, scrolling), 16550 serial logging, `kprintf`, `panic`, `ASSERT`, a crash report for every CPU exception with a full register dump, symbolised backtraces | 2, 3, 15 |
| **Interrupts and time** | Own GDT and IDT, 32 exception stubs, 8259 PIC remapped to vectors 32–47, 100 Hz PIT timer with uptime and `sleep_ms`, PS/2 keyboard with shift, caps lock and an optional Turkish Q layout | 3, 4 |
| **Kernel monitor** | Built-in prompt with `help`, `clear`, `uptime`, `mem`, `crash`, `reboot`, `ps`, `test`, `lsblk`, `hexdump` | 4, 5, 8, 12 |
| **Physical memory** | Multiboot memory map parser, bitmap page-frame allocator (single and contiguous frames), per-frame reference counts | 5, 11 |
| **Virtual memory** | Two-level paging, direct map of the first 512 MiB, VMM API (`vmm_map`, `vmm_unmap`, `vmm_virt_to_phys`), page-fault reports that explain every fault | 6 |
| **Kernel heap** | `kmalloc`, `kfree`, `kcalloc`, `krealloc` (first fit, splitting, coalescing, 16-byte alignment), magic numbers and `heap_check()`, optional slab cache | 7 |
| **Processes** | Kernel threads with guard pages, context switch in about ten lines of assembly, preemptive round-robin scheduler (optional MLFQ), idle task, task reaping | 8 |
| **Synchronisation** | Nesting `irq_save`/`irq_restore`, `xchg` spinlocks, wait queues without lost wake-ups, mutexes, counting semaphores, condition variables, a written lock order | 9 |
| **User mode** | TSS, ring 3, one page directory per process sharing the kernel half, ELF loader, `crt0`, every user pointer checked by `copy_from_user`/`copy_to_user` | 10 |
| **UNIX process model** | `fork` with copy-on-write, `execve` with `argv`/`envp`, `waitpid`, `exit` with zombies and orphan reparenting, lazy `brk`, stacks that grow on demand, `kill`, `SIGSEGV`, `SIGINT`, `init` as PID 1 | 11 |
| **Storage** | ATA PIO driver (polling, then interrupt-driven), block device layer, MBR partitions, RAM disk from a boot module, buffer cache with LRU replacement and write-back | 12 |
| **File systems** | VFS with mount points and path lookup, 32 descriptors per process, read-only root from a ustar archive, `devfs` (`/dev/tty`, `/dev/null`, `/dev/zero`, `/dev/hda`, `/dev/hdb`), writable **TurkFS** that Linux's `fsck.minix` accepts | 13 |
| **Userland** | Turk-libc (system call wrappers with `errno`, strings, `malloc`, buffered `stdio` with `printf`), pipes, terminal line discipline (echo, line editing, Ctrl+C, Ctrl+D), ANSI escape codes | 14 |
| **Shell** | `tsh`, the Turk shell: pipelines, `<`, `>`, `>>`, background jobs with `&`, `;`, quoting, built-ins `cd`, `pwd`, `exit`, `export`, `help`, `PATH` lookup | 14 |
| **Core utilities** | `echo`, `cat`, `wc`, `hexdump`, `ls`, `mkdir`, `rm`, `cp`, `mv`, `touch`, `ps`, `kill`, `sleep`, `uptime`, `clear`, `sync`, `shutdown` | 14 |
| **Security** | System call fuzzer, audited user/kernel boundary, `-fstack-protector-strong`, read-only kernel code, user and group IDs, file permissions, `/etc/passwd` and `login` | 15 |
| **Release** | Benchmark suite (null system call, context switch, process creation, file throughput, boot time), `make release`, bootable ISO, `v1.0` tag | 15 |

This is the Phase 14 milestone, taken from the plan's own tests:

```text
turk@os:/$ echo merhaba | wc -c
8
turk@os:/$ ls /bin > /mnt/list.txt
turk@os:/$ cat < /mnt/list.txt
...
turk@os:/$ sleep 5 &
turk@os:/$
```

---

## Technical decisions

These choices hold for the whole project. Each one follows from the books the plan is based on.

| Decision | Choice | Why |
|---|---|---|
| Target CPU | IA-32 (i386), 32-bit protected mode, BIOS boot | The practical guide, MINIX 3 and the Intel examples in the theory books are all 32-bit x86. Two-level paging is the simplest real MMU to learn. |
| Boot loader | GRUB through Multiboot 1; `qemu -kernel` for quick runs | Writing a boot loader is a detour. Phase 1 writes one 512-byte boot sector to learn how booting works, then GRUB takes over. |
| Languages | C (gnu11) and NASM (Intel syntax) | Assembly only where C can't work: the entry point, descriptor table loads, interrupt stubs, the context switch and entering user mode. Under 5 % of the code. |
| Toolchain | `i686-elf` cross GCC and binutils, GNU Make, Git | A cross compiler never assumes a Linux target, which prevents a whole family of confusing bugs. |
| Emulator and debugger | QEMU and GDB (remote stub); Bochs optional | QEMU is fast, scriptable and has a monitor for registers and page tables. |
| Kernel structure | Monolithic, modular source tree | Simplest to get right for a first kernel. MINIX's microkernel is studied as a contrast. |
| On-disk file system | MINIX V3 layout with 1 KiB blocks, named TurkFS | Fully described in *Operating Systems: Design and Implementation*, with source. Linux's `mkfs.minix -3` and `fsck.minix` create and check the images, and Linux can mount them. |
| System call ABI | `int 0x80`, Linux i386 call numbers | Any Linux reference table matches Turk-OS's. |
| Testing | In-kernel `ktest` suites, headless QEMU, `isa-debug-exit` | `make test` boots, runs every test and exits with a pass or fail code, so regressions show up at once. |

**Why monolithic when one of the books is about a microkernel?** In a monolithic kernel, drivers, the file system and
the scheduler all run in ring 0 and call each other as plain C functions. In a microkernel like MINIX 3 they run as
separate user-space processes that talk by message passing. Microkernels isolate faults better, but they need IPC,
servers and a protocol for every operation before anything works. For a first kernel, plain function calls keep the
path from idea to running code short. The MINIX code is still read in every phase, and Phase 15 offers an optional
experiment that moves one driver into a user-space server.

---

## Architecture

### Layers

| Layer | Components | Built in |
|---|---|---|
| **User space (ring 3)** | `init`, `tsh` shell, core utilities | P11, P14 |
| | Turk-libc: `stdio`, `malloc`, `string`, system call wrappers | P14 |
| | Your own ELF programs | P10, P11 |
| **System call interface** | `int 0x80`, argument checks, `copy_from_user` / `copy_to_user` | P10 |
| **Kernel (ring 0)** | Processes and scheduler: tasks, `fork`, `exec`, signals | P8, P11 |
| | Memory: PMM, VMM, heap, copy-on-write | P5–P7, P11 |
| | VFS: tarfs, TurkFS, devfs, pipes | P13, P14 |
| | Synchronisation: spinlocks, mutexes, semaphores, wait queues | P9 |
| | Drivers: VGA, serial, keyboard, PIT, tty, ATA, RAM disk, block layer, buffer cache | P2, P4, P12, P14 |
| | i386 layer: Multiboot entry, GDT, IDT and ISR stubs, PIC, TSS, paging, context switch | P1–P4, P6, P8, P10 |
| **Hardware (QEMU)** | CPU, RAM, VGA, 8259 PIC, 8254 PIT, 8042 keyboard controller, IDE disk, 16550 UART | |

### Boot sequence

1. **BIOS** (SeaBIOS in QEMU) finds a bootable device and hands control to GRUB.
2. **GRUB 2** reads `boot/grub.cfg`, loads `kernel.elf` at physical 1 MiB, loads any `module` lines (user programs,
   the root archive, the symbol table), and jumps to the kernel's physical entry point with `EAX = 0x2BADB002` and
   `EBX` pointing at the Multiboot information structure.
3. **`boot/loader.s`** starts with the Multiboot header (magic `0x1BADB002`, flags, checksum) in its own section,
   placed first by `link.ld`. Paging is still off, so it runs at physical addresses. It loads a boot page directory
   of 4 MiB pages that maps physical 0–4 MiB at 0 (temporarily) and physical 0–512 MiB at `0xC0000000`, enables
   `CR4.PSE`, `CR0.PG` and `CR0.WP`, and jumps into the higher half. There it drops the identity mapping, switches to
   its 16 KiB kernel stack and calls `kmain(magic, mbi)`.
4. **`kmain`** brings the system up in phase order: VGA and serial output, GDT and IDT, PIC, timer and keyboard,
   memory map and frame allocator, paging and the direct map, heap, scheduler, TSS and system calls, block devices,
   file systems.
5. **`init`** (PID 1) opens `/dev/tty` as descriptors 0, 1 and 2, starts `tsh` (after `login` from Phase 15), starts a
   new shell whenever one exits, and reaps orphaned processes.

With the kernel command line `test`, `kmain` runs every `ktest` suite instead and exits QEMU with the result.

### Virtual memory layout

Turk-OS uses the Linux-style 3 GiB / 1 GiB split. The kernel half is shared by every page directory, so interrupts
and system calls always find the kernel mapped.

| Virtual range | Size | Contents |
|---|---|---|
| `0x00000000`–`0x003FFFFF` | 4 MiB | Unmapped, so `NULL` and small offsets from it fault |
| `0x00400000`– | | Program image from the ELF file (`.text`, `.rodata`, `.data`, `.bss`), linked by `user/user.ld` |
| (above the image) | | User heap, growing up with `brk` (Phase 11) |
| ...–`0xBFFFFFFF` | | User stack, starting at `0xBFFFC000` and growing down on demand (Phase 11) |
| `0xC0000000`–`0xDFFFFFFF` | 512 MiB | **Direct map** of physical 0–512 MiB. Kernel image at `0xC0100000`, VGA buffer at `0xC00B8000`. `P2V`/`V2P` convert addresses. |
| `0xE0000000`–`0xEFFFFFFF` | 256 MiB | Kernel heap (Phase 7) |
| `0xF0000000`–`0xFFBFFFFF` | | Kernel stacks (with guard pages) and other kernel mappings |
| `0xFFC00000`–`0xFFFFFFFF` | 4 MiB | Reserved (for a recursive mapping, if ever wanted) |

Kernel page tables for `0xE0000000`–`0xFFBFFFFF` are allocated in advance, so a mapping made in one address space
shows up in all of them. Page-table entry bit 9 (`0x200`, one of the three bits free for the OS) marks copy-on-write
pages, and `CR0.WP` is set so the kernel obeys read-only pages too.

### System call interface

A user program loads the call number into `EAX` and up to three arguments into `EBX`, `ECX` and `EDX`, then executes
`int 0x80`. Vector `0x80` is a 32-bit trap gate with DPL 3, so ring 3 may use it and interrupts stay enabled during
the call. The kernel returns the result in `EAX`; errors come back as a negative `errno` value (for example
`-ENOSYS` for an unknown call), which Turk-libc turns into `-1` plus `errno`.

Turk-OS uses the **Linux i386 numbers**. The calls the plan introduces, by phase:

| No. | Call | Phase | | No. | Call | Phase |
|---:|---|:---:|---|---:|---|:---:|
| 1 | `exit` | 10 | | 36 | `sync` | 13 |
| 2 | `fork` | 11 | | 37 | `kill` | 11 |
| 3 | `read` | 10 | | 38 | `rename` | 14 |
| 4 | `write` | 10 | | 39 | `mkdir` | 13 |
| 5 | `open` | 13 | | 41 | `dup` | 13 |
| 6 | `close` | 13 | | 42 | `pipe` | 14 |
| 7 | `waitpid` | 11 | | 45 | `brk` | 11 |
| 10 | `unlink` | 13 | | 46 | `setgid` | 15 |
| 11 | `execve` | 11 | | 47 | `getgid` | 15 |
| 12 | `chdir` | 13 | | 54 | `ioctl` | 14 |
| 19 | `lseek` | 13 | | 63 | `dup2` | 13 |
| 20 | `getpid` | 10 | | 64 | `getppid` | 11 |
| 23 | `setuid` | 15 | | 106 | `stat` | 13 |
| 24 | `getuid` | 15 | | 108 | `fstat` | 13 |
| | | | | 141 | `getdents` | 13 |
| | | | | 158 | `sched_yield` | 10 |
| | | | | 162 | `nanosleep` (argument in milliseconds) | 10 |
| | | | | 183 | `getcwd` | 13 |

Phase 14 also adds a `getprocs` call for `ps`, which has no Linux equivalent. The full reference, with arguments,
return values and errors, goes into `docs/syscalls.md` in Phase 10.

### TurkFS on-disk format

TurkFS is the MINIX V3 file system with 1 KiB blocks, exactly as `mkfs.minix -3` lays it out. On an 8 MiB image:

| Block(s) | Region |
|---|---|
| 0 | Boot block (unused) |
| 1 | Superblock: inode and zone counts, bitmap sizes, first data zone, magic `0x4D5A`, block size 1024 |
| 2 | Inode bitmap |
| 3 | Zone bitmap |
| 4–174 | Inode table: 64-byte inodes, 16 per block (2736 inodes) |
| 175–8191 | Data zones: file contents, directories, indirect blocks |

- **Inode** (64 bytes): mode, link count, uid, gid, size, access/modify/change times, and ten zone pointers: seven
  direct, one single indirect (256 more zones), one double indirect (256 × 256 zones) and one unused triple indirect.
  Files larger than 263 KiB need the double indirect block.
- **Directory entry** (64 bytes): a 32-bit inode number and a 60-byte name. Inode 0 marks an empty slot.
- **Bitmaps:** bit *n* of the inode map is inode *n*; bit *n* of the zone map is zone *n* + `s_firstdatazone` − 1;
  bit 0 of both is reserved.
- **Crash safety:** writes are ordered as allocate, initialise, then link, so a crash leaves at worst leaked blocks.
  `fsck.minix -f` on the host checks the result.

---

## Roadmap

Sixteen phases in four stages. Each phase depends on everything before it and has its own PDF in
[`docs/pdf/`](docs/pdf/).

```mermaid
flowchart LR
    subgraph S1["Stage I · Bare metal (0–28 %)"]
        P0["P0 Foundations"] --> P1["P1 Assembly & boot"] --> P2["P2 Getting to C"] --> P3["P3 GDT & IDT"] --> P4["P4 Hardware IRQs"]
    end
    subgraph S2["Stage II · Memory (28–48 %)"]
        P5["P5 Physical memory"] --> P6["P6 Paging"] --> P7["P7 Kernel heap"]
    end
    subgraph S3["Stage III · Processes and protection (48–77 %)"]
        P8["P8 Processes"] --> P9["P9 Synchronization"] --> P10["P10 User mode"] --> P11["P11 Process API"]
    end
    subgraph S4["Stage IV · Persistence and userland (77–100 %)"]
        P12["P12 Storage"] --> P13["P13 File systems"] --> P14["P14 Userland"] --> P15["P15 Turk-OS 1.0"]
    end
    P4 --> P5
    P7 --> P8
    P11 --> P12
```

| # | Phase | You finish with | Weeks | Difficulty | Progress | Document |
|---:|---|---|:---:|:---:|:---:|---|
| 0 | Foundations and toolchain | Cross compiler, QEMU, GDB and a Git repo; C warm-up library with unit tests | 2–3 | ★☆☆☆☆ | 0 → 5 % | [PDF](docs/pdf/phase-00-foundations.pdf) |
| 1 | x86 assembly and booting | A 512-byte boot sector; a Multiboot kernel that GRUB and QEMU boot | 2–3 | ★★★☆☆ | 5 → 10 % | [PDF](docs/pdf/phase-01-assembly-and-boot.pdf) |
| 2 | Getting to C: screen, serial, kprintf | VGA text driver, serial logging, `kprintf`, `panic` | 1–2 | ★★☆☆☆ | 10 → 16 % | [PDF](docs/pdf/phase-02-output-drivers.pdf) |
| 3 | Segmentation, IDT and exceptions | Own GDT and IDT; register dump on every CPU exception | 2 | ★★★★☆ | 16 → 22 % | [PDF](docs/pdf/phase-03-gdt-idt-exceptions.pdf) |
| 4 | Hardware interrupts | PIC, 100 Hz timer, keyboard driver, kernel command monitor | 2 | ★★★☆☆ | 22 → 28 % | [PDF](docs/pdf/phase-04-hardware-interrupts.pdf) |
| 5 | Physical memory management | Memory map parsing, bitmap page-frame allocator, first kernel tests | 1–2 | ★★★☆☆ | 28 → 34 % | [PDF](docs/pdf/phase-05-physical-memory.pdf) |
| 6 | Paging and the higher-half kernel | Paging on, kernel at `0xC0000000`, VMM API, page-fault reports | 2–3 | ★★★★★ | 34 → 42 % | [PDF](docs/pdf/phase-06-paging-higher-half.pdf) |
| 7 | Kernel heap and data structures | `kmalloc`/`kfree`, intrusive lists, `make test` in headless QEMU | 1–2 | ★★★☆☆ | 42 → 48 % | [PDF](docs/pdf/phase-07-kernel-heap.pdf) |
| 8 | Processes and scheduling | Kernel threads, context switch, preemptive round-robin, sleep | 3 | ★★★★★ | 48 → 56 % | [PDF](docs/pdf/phase-08-processes-scheduling.pdf) |
| 9 | Synchronization | Spinlocks, wait queues, mutexes, semaphores; blocking keyboard input | 2 | ★★★★☆ | 56 → 62 % | [PDF](docs/pdf/phase-09-synchronization.pdf) |
| 10 | User mode and system calls | TSS, ring 3, `int 0x80`, ELF loader, first user programs | 3 | ★★★★★ | 62 → 70 % | [PDF](docs/pdf/phase-10-user-mode-syscalls.pdf) |
| 11 | Process API | `fork` (copy-on-write), `exec`, `wait`, `exit`, `brk`, basic signals | 3 | ★★★★★ | 70 → 77 % | [PDF](docs/pdf/phase-11-process-api.pdf) |
| 12 | Storage | ATA PIO driver, block layer, MBR partitions, buffer cache | 2 | ★★★★☆ | 77 → 83 % | [PDF](docs/pdf/phase-12-storage-drivers.pdf) |
| 13 | File systems | VFS, file descriptors, tar initrd, writable TurkFS | 3–4 | ★★★★★ | 83 → 90 % | [PDF](docs/pdf/phase-13-file-systems.pdf) |
| 14 | Userland | Turk-libc, pipes, tty line discipline, `tsh` shell, coreutils | 3 | ★★★★☆ | 90 → 96 % | [PDF](docs/pdf/phase-14-userland-shell.pdf) |
| 15 | Hardening and release | Fuzzing, permissions, benchmarks, docs, v1.0 ISO; optional tracks | 3–4+ | ★★★★☆ | 96 → 100 % | [PDF](docs/pdf/phase-15-hardening-release.pdf) |

The [roadmap PDF](docs/pdf/00-roadmap.pdf) has the full picture: phase map, calendar timeline, risk table and the
getting-unstuck protocol.

**After 1.0**, Phase 15 offers optional tracks that can be combined:

| Track | What it adds |
|---|---|
| A. Graphics and Turkish text | Linear framebuffer, a PSF font with the full Turkish alphabet (ğ, ş, ı, İ, Ğ, Ş), PS/2 mouse, a simple window system |
| B. Multiprocessor (SMP) | ACPI MADT, local and I/O APIC, starting the other CPUs, per-CPU run queues, TLB shootdown |
| C. Networking | PCI, an e1000 or RTL8139 driver, Ethernet, ARP, IPv4, ICMP, UDP, minimal TCP, sockets |
| D. 64-bit port | Long mode, four-level paging, Multiboot2, `syscall`/`sysret` |
| E. RISC-V port | Separate architecture layer, Sv32 paging, QEMU `virt`, then a self-designed RV32IMC core |
| F. Microkernel experiment | The ATA driver or TurkFS as a user-space server with message passing, and its measured cost |
| G. Virtualisation | What QEMU and KVM do, running with `-enable-kvm`, comparing benchmarks |

---

## Repository layout

The folders fill up phase by phase. Empty folders hold a `.gitkeep` file so Git tracks them; delete it once the
folder has real files.

```text
Turk-OS/
├── README.md               this file
├── CHANGELOG.md            what changed at each milestone
├── LICENSE                 MIT License
├── Makefile                kernel build: all, run, iso, debug, clean, test, release     (Phase 1+)
├── link.ld                 kernel linker script, higher half from Phase 6               (Phase 1)
├── boot/                   loader.s (Multiboot entry), grub.cfg                         (Phase 1)
├── kernel/
│   ├── kmain.c             kernel entry point in C                                      (Phase 2)
│   ├── arch/i386/          gdt.c idt.c isr.s irq.c pic.c pit.c tss.c paging.c switch.s  (Phases 3, 4, 6, 8, 10)
│   ├── mm/                 pmm.c vmm.c heap.c                                           (Phases 5–7)
│   ├── proc/               task.c sched.c fork.c exec.c signal.c                        (Phases 8, 11)
│   ├── sync/               spinlock.c mutex.c semaphore.c waitq.c                       (Phase 9)
│   ├── fs/                 vfs.c file.c tarfs.c turkfs.c devfs.c pipe.c                 (Phases 13, 14)
│   ├── drivers/            vga.c serial.c keyboard.c tty.c ata.c ramdisk.c
│   │                       blockdev.c bcache.c                                          (Phases 2, 4, 12, 14)
│   └── syscall/            syscall.c uaccess.c                                          (Phase 10)
├── lib/                    libk: string.c convert.c printf.c bitmap.c ringbuf.c list.h  (Phase 0+)
├── include/                kernel headers, including locks.h with the lock order        (Phase 0+)
├── user/
│   ├── user.ld             user program linker script (programs start at 0x00400000)    (Phase 10)
│   ├── libc/               Turk-libc                                                    (Phase 14)
│   ├── init/               init, PID 1                                                  (Phase 11)
│   ├── sh/                 tsh, the Turk shell                                          (Phase 14)
│   └── bin/                core utilities, fuzz, bench                                  (Phases 14, 15)
├── rootfs/                 files for the root archive (/etc/motd, /etc/passwd); bin/ is generated  (Phase 13)
├── tests/                  host tests for lib/, in-kernel ktest suites                  (Phases 0, 5, 7)
├── tools/                  mkturkfs.c and helper scripts
└── docs/                   study plan (LaTeX + PDFs), setup guide, lab journal           (see docs/README.md)
```

Generated files never enter Git. Everything the build produces goes to `build/`: `kernel.elf`, `boot.bin`,
`turkos.iso` and its `iso/` staging tree, `disk.img`, `turkfs.img`, `initrd.tar`, user programs under `build/user/`,
and `serial.log`. See [`.gitignore`](.gitignore) for the full list.

---

## Getting started

> **Status:** the repository currently holds the study plan and the empty source tree. The top-level `Makefile`
> arrives in Phase 1, so until then the only thing to build is the documentation (`make -C docs`).

### Host requirements

Turk-OS builds on **Arch Linux, Fedora, Ubuntu/Debian, macOS and Windows (through WSL2)**. Every platform uses the
same tools; only package names and a few command names differ. The full,
step-by-step guide (packages, the cross compiler, editor setup, verification and troubleshooting for each platform)
is **[docs/setup.md](docs/setup.md)**.

| Platform | Install the host tools | Cross compiler |
|---|---|---|
| **Arch Linux** | `sudo pacman -S --needed base-devel git nasm qemu-desktop gdb libisoburn mtools grub` | [Build from source](docs/setup.md#3-build-the-i686-elf-cross-compiler), or `i686-elf-gcc` from the AUR |
| **Fedora** | `sudo dnf install gcc make git nasm qemu-system-x86 qemu-img gdb xorriso mtools grub2-tools grub2-tools-extra grub2-pc-modules util-linux` | Build from source |
| **Ubuntu / Debian** | `sudo apt install build-essential git curl nasm qemu-system-x86 qemu-system-gui qemu-utils gdb xorriso mtools grub-pc-bin grub-common util-linux` | Build from source |
| **macOS** | `brew install i686-elf-binutils i686-elf-gcc i686-elf-grub i386-elf-gdb nasm qemu xorriso mtools util-linux make coreutils` ([details](docs/setup.md#macos)) | Included in the `brew` line |
| **Windows 10 / 11** | `wsl --install -d Ubuntu`, then the Ubuntu line inside WSL2 ([details](docs/setup.md#windows-wsl2)) | Build from source inside WSL2 |

Building the cross compiler from source also needs Bison, Flex, Texinfo and the GMP, MPFR and MPC development
packages; [setup.md](docs/setup.md#2-install-the-host-tools) has the exact line for each platform. The result is an
`i686-elf` GCC and binutils in `~/opt/cross/bin`, on your `PATH`.

| Tool | Used for | Check |
|---|---|---|
| `i686-elf-gcc`, `i686-elf-ld` | Compiling and linking the kernel and user programs | `i686-elf-gcc --version` |
| `nasm` | Boot sector, loader, interrupt stubs, context switch | `nasm -v` |
| `qemu-system-i386`, `qemu-img` | Running and testing the OS; disk images | `qemu-system-i386 --version` |
| `gdb` (`i386-elf-gdb` on macOS) | Source-level kernel debugging over QEMU's GDB stub | `gdb --version` |
| `grub-mkrescue` (Arch, Ubuntu/Debian), `grub2-mkrescue` (Fedora) or `i686-elf-grub-mkrescue` (macOS), with `xorriso` and `mtools` | Building the bootable ISO | `grub-mkrescue --version` |
| `mkfs.minix`, `fsck.minix` (util-linux) | Creating and checking TurkFS images | `fsck.minix --version` |
| `latexmk` + TeX Live | Rebuilding the study plan PDFs (optional) | `latexmk -v` |

Command names that differ between platforms are listed in
[setup.md, section 4](docs/setup.md#4-tool-names-on-each-platform), together with a Makefile snippet that finds the
right one automatically.

### Build and run

```sh
git clone https://github.com/hikmethankolay/Turk-OS.git
cd Turk-OS
make run
```

The `Makefile` grows target by target as the phases need them:

| Command | What it does | From |
|---|---|:---:|
| `make` / `make all` | Builds `build/kernel.elf` from `boot/`, `kernel/` and `lib/` | Phase 1 |
| `make run` | Boots the kernel with `qemu-system-i386 -kernel`; from Phase 2 the serial port goes to your terminal (`-serial stdio`) | Phase 1 |
| `make iso` | Builds `build/turkos.iso` with `grub-mkrescue` (`grub2-mkrescue` on Fedora) | Phase 1 |
| `make debug` | Starts QEMU frozen with its GDB stub (`-s -S`) and attaches GDB with a breakpoint at the entry point | Phase 1 |
| `make clean` | Deletes `build/` | Phase 1 |
| `make test` | Boots a headless QEMU with the kernel command line `test`, runs every test, and fails if any test fails | Phase 7 |
| `make release` | Builds the kernel, user programs, root archive, a populated TurkFS image and a GRUB ISO | Phase 15 |
| `make -C docs` | Rebuilds the study plan PDFs (see [docs/README.md](docs/README.md)) | now |

### Running QEMU by hand

```sh
# Quick boot through QEMU's built-in Multiboot loader, serial log in the terminal
qemu-system-i386 -kernel build/kernel.elf -serial stdio

# Boot exactly like a real PC: BIOS -> GRUB -> kernel
qemu-system-i386 -cdrom build/turkos.iso -serial stdio

# Choose the amount of RAM (the memory map must match)
qemu-system-i386 -kernel build/kernel.elf -serial stdio -m 128

# Keep the serial log in a file instead
qemu-system-i386 -kernel build/kernel.elf -serial file:build/serial.log

# Pass user programs as Multiboot modules (Phase 10)
qemu-system-i386 -kernel build/kernel.elf -serial stdio \
    -initrd "build/user/hello.elf,build/user/evil.elf"

# Attach a scratch disk as primary master (Phase 12) and TurkFS as primary slave (Phase 13)
qemu-img create -f raw build/disk.img 64M
dd if=/dev/zero of=build/turkfs.img bs=1024 count=32768 && mkfs.minix -3 build/turkfs.img
qemu-system-i386 -kernel build/kernel.elf -serial stdio \
    -drive file=build/disk.img,format=raw,if=ide,index=0,media=disk \
    -drive file=build/turkfs.img,format=raw,if=ide,index=1,media=disk
```

### Debugging

Kernel bugs often end in a silent reboot (a triple fault). These tools turn that into something you can read:

| Tool | How | What it shows |
|---|---|---|
| Serial log | `-serial stdio`, and a `kprintf` at every boot step | How far the kernel got before it died |
| Interrupt log | `-d int,cpu_reset -no-reboot` | Every interrupt and exception with its vector and registers; QEMU stops instead of rebooting |
| QEMU monitor | `-monitor stdio`, or `Ctrl+Alt+2` in the QEMU window | `info registers` (GDT, IDT, CR0, CR3), `info mem` (page mappings), `info block` (disks) |
| GDB | `make debug` | Breakpoints, `stepi`, `info registers`, `x/4xw $esp`, `p/x $eax`, `layout asm` |
| Crash report | Phase 3 exception handler | Exception name, error code and a full register dump instead of a reboot |
| Page-fault report | Phase 6 fault handler | The faulting address from CR2 and a decoded error code: read or write, user or kernel, not present or protection |
| Backtrace | Phase 15 symbol table module | Function names in every panic |
| Host checks | `grub-file --is-x86-multiboot build/kernel.elf` (`grub2-file` on Fedora), `i686-elf-objdump -d -M intel`, `i686-elf-readelf -S`, `hexdump -C` | Whether the image is what you think it is |

**When stuck:** reproduce the bug reliably and shrink it; look instead of guessing (serial log, `-d int`, monitor,
GDB); write the hypothesis in the [lab journal](docs/journal.md) before changing code, one change at a time;
re-read the exact rule in the Intel manual or the book; compare with how MINIX or xv6 does the same step; after two
sessions without progress, ask for help and bring the journal notes. Small commits let `git bisect` find the change
that broke the kernel.

### Testing

| Level | What | From |
|---|---|:---:|
| Host unit tests | `lib/` compiled with the host GCC and run with AddressSanitizer and UBSan: `gcc -std=gnu11 -Wall -Wextra -g -fsanitize=address,undefined -Iinclude tests/test_lib.c lib/*.c -o build/test_lib && ./build/test_lib` | Phase 0 |
| Kernel tests | A small `ktest` framework; the monitor's `test` command runs every registered test | Phase 5 |
| Automated runs | `make test`: headless QEMU, results on serial, exit code through `isa-debug-exit` (QEMU exits with 2v+1, so 1 means pass), with a 120-second timeout | Phase 7 |
| Stress tests | 10 000-step heap test, 1000 task and fork cycles without leaks, bounded buffer and dining philosophers | Phases 7–11 |
| Evil programs | Seven user programs that misbehave on purpose; each must be killed while the kernel survives | Phase 10 |
| File system checks | `fsck.minix -f build/turkfs.img` on the host after Turk-OS has written to it | Phase 13 |
| Shell tests | `tsh -c` command lines compared with expected output, part of `make test` | Phase 14 |
| Fuzzing | Four system call fuzzers plus a stress script for one hour, with no panic and no leak | Phase 15 |

---

## The study plan

The plan in [`docs/`](docs/) is the backbone of the project: a roadmap and one document per phase, written in LaTeX
and committed as PDFs. See **[docs/README.md](docs/README.md)** for the full index and how the documents are built.

Every phase document has the same layout: an *at a glance* box (duration, difficulty, prerequisites, result),
*why this phase*, *concepts you need*, a *reading plan* with printed page numbers marked **REQUIRED**,
**RECOMMENDED** or **OPTIONAL**, a *MINIX lens* box, numbered *build it* tasks each with a **Verify** step, a
debugging table, the *milestone* checklist, *self-check questions* and *going further*.

The study loop, repeated in every phase:

1. **Read the theory first**, usually OSTEP, then the REQUIRED items from the other books.
2. **Read the practical chapter** in *The little book about OS development*, when there is one.
3. **Look at a real system**: the MINIX lens box points to the matching section and source file.
4. **Build in small steps**: one task at a time, boot after each change, commit when it works.
5. **Verify**: a task isn't finished until its Verify step passes.
6. **Write it down** in the [lab journal](docs/journal.md): what you tried, what broke and why.
7. **Answer the self-check questions** without looking. If you can't, go back to the reading.

Never start a phase while the previous milestone is still shaky; every later phase builds on it.

---

## Books and references

The books are **not** part of this repository, so get your own copies before Phase 0. Two are free online from their
authors. The other three are copyrighted textbooks: buy them from the publisher or a bookshop (a used copy is fine),
or borrow them from a library.

| Code | Book | Where to get it | Role in the plan |
|---|---|---|---|
| **LOB** | Erik Helin, Adam Renberg. *The little book about OS development*, 2015. | Free at [littleosbook.github.io](https://littleosbook.github.io/), as a web page or a PDF. The page numbers in the plan refer to the PDF. | The practical spine: its 14 chapters give the build order for Phases 1–14. Uses GRUB Legacy and Bochs; the phase docs give the QEMU and GRUB 2 equivalents. |
| **OSTEP** | Remzi H. and Andrea C. Arpaci-Dusseau. *Operating Systems: Three Easy Pieces*, v0.90, Arpaci-Dusseau Books, 2015. | Free at [pages.cs.wisc.edu/~remzi/OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/), one PDF per chapter (current version 1.10). Printed copies are sold there too. | Main theory text, the first read in every phase. |
| **OSC** | A. Silberschatz, P. B. Galvin, G. Gagne. *Operating System Concepts*, 10th ed., Wiley, 2018. ISBN 978-1-119-32091-3. | Wiley, a bookshop or a library. | Encyclopedic reference; a second explanation when an OSTEP chapter didn't click. |
| **MOS** | Andrew S. Tanenbaum, Herbert Bos. *Modern Operating Systems*, 4th ed., Pearson, 2015. ISBN 978-0-13-359162-0. | Pearson, a bookshop or a library. | Strongest on hardware, I/O, clocks, disks, security and OS design (Ch. 12). |
| **OSDI** | Andrew S. Tanenbaum, Albert S. Woodhull. *Operating Systems: Design and Implementation*, 3rd ed., Pearson Prentice Hall, 2006. ISBN 978-0-13-142938-3. | A bookshop (often second-hand) or a library. | MINIX 3 printed line by line: how a real x86 kernel does each step. |

Page numbers in the plan are the *printed* page numbers of these editions, not PDF page indices. With any other
edition, find each reading by the chapter and section numbers that every reading-plan row gives. For OSTEP this is
the normal case: each free chapter PDF starts at page 1, and version 1.10 is paginated differently from v0.90.
Chapters 1–43, and every section the plan cites, have the same numbers in both versions; only chapter 23 was renamed
(*Complete Virtual Memory Systems*, formerly *The VAX/VMS Virtual Memory System*). MINIX references such as
`kernel/proc.c (07400)` give the file and its starting line in OSDI Appendix B.

Hardware facts come from outside the books:

- **Intel 64 and IA-32 Architectures Software Developer's Manual, Volume 3A** (System Programming Guide), chapters
  2–7: protection, segmentation, paging, interrupts and the TSS.
  [intel.com/sdm](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- **OSDev Wiki**: practical pages on nearly every device in this plan, plus lists of common mistakes.
  [wiki.osdev.org](https://wiki.osdev.org)
- **Multiboot Specification 0.6.96**: the contract between GRUB and the kernel.
  [gnu.org](https://www.gnu.org/software/grub/manual/multiboot/multiboot.html)
- **NASM manual**: assembler syntax. [nasm.us](https://www.nasm.us/doc/)
- **xv6** (optional): MIT's small UNIX-like teaching kernel, which OSTEP refers to.
  [github.com/mit-pdos/xv6-public](https://github.com/mit-pdos/xv6-public)

---

## Conventions

**C**
- `-std=gnu11 -ffreestanding`, compiled with `i686-elf-gcc`, never the host compiler (except for host tests in
  `tests/`). Kernel flags: `-O2 -g -Wall -Wextra -fno-omit-frame-pointer -Iinclude`; link with `-nostdlib -lgcc`.
- Only the freestanding headers are available: `stdint.h`, `stddef.h`, `stdbool.h`, `stdarg.h`, `limits.h`.
  Everything else, `memcpy` included, lives in `lib/`.
- Hardware-defined structures are `__attribute__((packed))` and their sizes are checked with `_Static_assert`.
- Memory-mapped device registers are accessed through `volatile` pointers.

**Assembly**
- NASM, Intel syntax, `.s` files assembled with `nasm -f elf32`. Read compiler output with `objdump -d -M intel` so it
  matches.
- Assembly only where C can't work: the entry point, descriptor table loads, interrupt stubs, the context switch and
  the jump to user mode. Functions called from C follow cdecl and preserve `EBX`, `ESI`, `EDI`, `EBP` and `ESP`.

**Kernel design rules**
- Every user pointer goes through `copy_from_user`, `copy_to_user` or `strncpy_from_user`. Copy first, then check the
  copy.
- Every shared structure has a documented lock, and locks are taken in the order written in `include/locks.h`.
- Every resource has a limit: processes (64), descriptors (32 per process), pipe buffers, path lengths.
- Every boot step logs to the serial port.

**Git**
- Commit small and often, and only when the kernel boots. Small commits make `git bisect` useful.
- Line endings are LF everywhere (enforced by [`.gitattributes`](.gitattributes)); Makefile recipe lines start with a
  real Tab.
- Build output stays in `build/` and out of Git. The study plan PDFs in `docs/pdf/` are committed on purpose.
- Record each finished milestone in [CHANGELOG.md](CHANGELOG.md) and tick it in the tracker below. Version 1.0 is
  tagged `v1.0` at the end of Phase 15.

---

## Milestone tracker

A phase is done only when all of its exit criteria are met.

- [ ] **P0** — `i686-elf-gcc`, NASM, QEMU and GDB work; repo created; C warm-up library passes its tests.
- [ ] **P1** — Boot sector prints a message; Multiboot kernel boots via `-kernel` and via a GRUB ISO.
- [ ] **P2** — Coloured banner on screen, same log on the serial port, `kprintf` and `panic` tested.
- [ ] **P3** — Own GDT and IDT loaded; divide-by-zero shows a register dump; `int3` returns cleanly.
- [ ] **P4** — Timer ticks at 100 Hz; keyboard input with shift and caps; monitor commands run.
- [ ] **P5** — Memory map printed; frame allocator passes its tests from 16 MiB to 3 GiB of RAM.
- [ ] **P6** — Kernel runs at `0xC0000000`; null dereference produces a clean page-fault report.
- [ ] **P7** — 10 000-operation heap stress test passes; `make test` runs headless.
- [ ] **P8** — Several kernel threads preempt each other; `sleep` works; no leaks over 1000 task cycles.
- [ ] **P9** — Race demo fixed by locks; bounded buffer and dining philosophers run without deadlock.
- [ ] **P10** — Ring-3 program prints through `write`; faulty programs are killed, the kernel survives.
- [ ] **P11** — `fork`/`exec`/`wait` chain works; copy-on-write verified; no leaks over 1000 forks.
- [ ] **P12** — QEMU disk identified; data survives reboot; partitions detected; buffer cache in use.
- [ ] **P13** — Files written by Turk-OS pass `fsck.minix` on the host; file descriptor semantics tested.
- [ ] **P14** — Boots to the `tsh` prompt; pipes, redirection and Ctrl+C work; coreutils present.
- [ ] **P15** — One hour of fuzzing without a panic; benchmarks and docs written; v1.0 tagged.

---

## License

Turk-OS is released under the [MIT License](LICENSE), © 2026 Hikmethan KOLAY. That covers the source code and the study plan in `docs/`.
