# Turk-OS

A small UNIX-like operating system for 32-bit x86 PCs, built from scratch as a study project. It boots with GRUB
(Multiboot 1), is written in C (gnu11) and NASM, and runs in QEMU.

## Study plan

The plan lives in [docs/](docs/). Start with [docs/pdf/00-roadmap.pdf](docs/pdf/00-roadmap.pdf), then work through
one PDF per phase, 0 to 15. To rebuild the PDFs from the LaTeX sources, run `make -C docs`.

## Layout

The folders fill up phase by phase. Each phase document says which parts it adds.

```
Turk-OS/
|-- docs/                  study plan (LaTeX), journal.md, architecture notes
|-- boot/                  loader.s (Multiboot entry), grub.cfg
|-- kernel/
|   |-- arch/i386/         GDT, IDT, ISR stubs, PIC, PIT, TSS, paging, context switch
|   |-- mm/                physical memory, virtual memory, heap
|   |-- proc/              tasks, scheduler, fork, exec, signals
|   |-- sync/              spinlocks, mutexes, semaphores, wait queues
|   |-- fs/                VFS, tarfs, TurkFS, devfs, pipes
|   |-- drivers/           VGA, serial, keyboard, tty, ATA, RAM disk, block layer, buffer cache
|   `-- syscall/           system call table, user memory access
|-- lib/                   libk: string, printf, bitmap, ring buffer, lists
|-- include/               kernel headers
|-- user/
|   |-- libc/              Turk-libc
|   `-- init/ sh/ bin/     init, tsh shell, coreutils
|-- tests/                 ktest suites
`-- tools/                 mkturkfs and helper scripts
```

`Books/` holds the reference books locally and is not tracked by Git. Build output goes to `build/`, which Git ignores.
