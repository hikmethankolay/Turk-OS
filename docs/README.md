# Turk-OS documentation

This folder holds the **Turk-OS study plan**, a roadmap and one document per phase that take the project from an empty
folder to Turk-OS 1.0. It also holds the working documents that the plan asks you to keep while building.
For the project overview, see the [main README](../README.md).

## Contents

1. [Documents at a glance](#documents-at-a-glance)
2. [The study plan](#the-study-plan)
3. [How a phase document is organised](#how-a-phase-document-is-organised)
4. [Reading the reading plans](#reading-the-reading-plans)
5. [Working documents](#working-documents)
6. [Building the PDFs](#building-the-pdfs)
7. [Editing or adding a document](#editing-or-adding-a-document)

---

## Documents at a glance

| File | What it is |
|---|---|
| [`setup.md`](setup.md) | Development environment setup for Arch, Fedora, Ubuntu/Debian, macOS and Windows (WSL2): host packages, the cross compiler, editor, verification |
| [`journal.md`](journal.md) | The lab journal: one entry per work session (started in Phase 0) |
| [`pdf/`](pdf/) | The study plan as PDFs, ready to read (committed to Git) |
| [`00-roadmap.tex`](00-roadmap.tex) | Source of the roadmap |
| [`phases/`](phases/) | Sources of the sixteen phase documents |
| [`common/turkos.sty`](common/turkos.sty) | Shared LaTeX style: colours, title banner, boxes, tables, code listings |
| [`Makefile`](Makefile), [`.latexmkrc`](.latexmkrc) | Build rules for the PDFs |

---

## The study plan

Start with the roadmap, then read one phase at a time. Don't start a phase until the previous milestone is fully met.

| Document | Stage | Topic | Weeks | Difficulty | Progress |
|---|---|---|:---:|:---:|:---:|
| [Roadmap](pdf/00-roadmap.pdf) · [source](00-roadmap.tex) | | How to use the plan, the five books, technical decisions, phase map, timeline, final architecture, risks, master tracker | | | |
| [Phase 0](pdf/phase-00-foundations.pdf) · [source](phases/phase-00-foundations.tex) | I · Bare metal | Foundations and toolchain | 2–3 | 1/5 | 0 → 5 % |
| [Phase 1](pdf/phase-01-assembly-and-boot.pdf) · [source](phases/phase-01-assembly-and-boot.tex) | I · Bare metal | x86 assembly and the boot process | 2–3 | 3/5 | 5 → 10 % |
| [Phase 2](pdf/phase-02-output-drivers.pdf) · [source](phases/phase-02-output-drivers.tex) | I · Bare metal | Getting to C: screen, serial and `kprintf` | 1–2 | 2/5 | 10 → 16 % |
| [Phase 3](pdf/phase-03-gdt-idt-exceptions.pdf) · [source](phases/phase-03-gdt-idt-exceptions.tex) | I · Bare metal | Segmentation, the IDT and CPU exceptions | 2 | 4/5 | 16 → 22 % |
| [Phase 4](pdf/phase-04-hardware-interrupts.pdf) · [source](phases/phase-04-hardware-interrupts.tex) | I · Bare metal | Hardware interrupts: PIC, timer and keyboard | 2 | 3/5 | 22 → 28 % |
| [Phase 5](pdf/phase-05-physical-memory.pdf) · [source](phases/phase-05-physical-memory.tex) | II · Memory | Physical memory management | 1–2 | 3/5 | 28 → 34 % |
| [Phase 6](pdf/phase-06-paging-higher-half.pdf) · [source](phases/phase-06-paging-higher-half.tex) | II · Memory | Paging and the higher-half kernel | 2–3 | 5/5 | 34 → 42 % |
| [Phase 7](pdf/phase-07-kernel-heap.pdf) · [source](phases/phase-07-kernel-heap.tex) | II · Memory | Kernel heap and core data structures | 1–2 | 3/5 | 42 → 48 % |
| [Phase 8](pdf/phase-08-processes-scheduling.pdf) · [source](phases/phase-08-processes-scheduling.tex) | III · Processes | Processes, context switching and scheduling | 3 | 5/5 | 48 → 56 % |
| [Phase 9](pdf/phase-09-synchronization.pdf) · [source](phases/phase-09-synchronization.tex) | III · Processes | Synchronization and concurrency in the kernel | 2 | 4/5 | 56 → 62 % |
| [Phase 10](pdf/phase-10-user-mode-syscalls.pdf) · [source](phases/phase-10-user-mode-syscalls.tex) | III · Processes | User mode and system calls | 3 | 5/5 | 62 → 70 % |
| [Phase 11](pdf/phase-11-process-api.pdf) · [source](phases/phase-11-process-api.tex) | III · Processes | Process API: `fork`, `exec`, `wait` and `exit` | 3 | 5/5 | 70 → 77 % |
| [Phase 12](pdf/phase-12-storage-drivers.pdf) · [source](phases/phase-12-storage-drivers.tex) | IV · Persistence | Storage: block devices, ATA driver and buffer cache | 2 | 4/5 | 77 → 83 % |
| [Phase 13](pdf/phase-13-file-systems.pdf) · [source](phases/phase-13-file-systems.tex) | IV · Persistence | File systems: VFS, initrd and TurkFS | 3–4 | 5/5 | 83 → 90 % |
| [Phase 14](pdf/phase-14-userland-shell.pdf) · [source](phases/phase-14-userland-shell.tex) | IV · Persistence | Userland: libc, shell and utilities | 3 | 4/5 | 90 → 96 % |
| [Phase 15](pdf/phase-15-hardening-release.pdf) · [source](phases/phase-15-hardening-release.tex) | IV · Persistence | Hardening, polish and release: Turk-OS 1.0 | 3–4+ | 4/5 | 96 → 100 % |

The progress column estimates how much of the whole journey is behind you when that phase's milestone is met.
Durations assume 10–12 hours a week, about 43 weeks in total.

### What each phase covers

<details>
<summary><b>Stage I: Bare metal (Phases 0–4)</b>. The tools, enough assembly, how a PC boots, output, exceptions and interrupts.</summary>

| Phase | Tasks |
|---|---|
| **0** Foundations | Install the host tools · make the editor's terminal see them · build the `i686-elf` cross compiler · create the repository skeleton · write and test the C warm-up library (`string`, `convert`, `bitmap`, `ringbuf`, `list.h`) · read your first assembly · smoke-test QEMU and GDB · start the lab journal |
| **1** Assembly and boot | Read compiler output · write assembly that C can call (`max`, `sum_array`) · a 512-byte boot sector that prints "Merhaba, Turk-OS!" · "Hello Cafebabe": a Multiboot kernel that puts `0xCAFEBABE` in `EAX` · boot through a real GRUB ISO · a Makefile (`all`, `run`, `iso`, `debug`, `clean`) · your first kernel debugging session |
| **2** Getting to C | Give the loader a stack and call C · compile C for the kernel · port I/O helpers · the VGA text driver · the serial driver · `kprintf` and the console layer · `panic` and `ASSERT` · read the Multiboot information |
| **3** GDT, IDT, exceptions | Describe the GDT in C · load it and reload the segment registers · build and load the IDT · write the interrupt stubs · the C exception handler · a handler registration API · trigger every exception you can |
| **4** Hardware interrupts | Remap and mask the PIC · IRQ stubs and dispatch · the timer · the keyboard driver · the Turkish Q layout (optional) · line input and the kernel monitor |

</details>

<details>
<summary><b>Stage II: Memory (Phases 5–7)</b>. Which RAM exists, paging, the higher half, the heap and the test harness.</summary>

| Phase | Tasks |
|---|---|
| **5** Physical memory | Walk the memory map · know where the kernel ends · the bitmap allocator · a tiny test framework · test the allocator hard · a real `mem` command |
| **6** Paging | Warm-up: identity paging from C · a page-fault handler that explains itself · move the kernel to the higher half · fix every address that assumed physical = virtual · the VMM: mapping 4 KiB pages · pre-allocate the kernel's page tables · test the fault paths deliberately |
| **7** Kernel heap | Whole pages from the direct map · a heap region that grows · `kmalloc` and `kfree` · the rest of the API and a heap checker · an intrusive list · `make test` · stress the heap · a slab cache (optional) |

</details>

<details>
<summary><b>Stage III: Processes and protection (Phases 8–11)</b>. Multitasking, locks, ring 3, system calls and the UNIX process model.</summary>

| Phase | Tasks |
|---|---|
| **8** Processes | The task structure · kernel stacks with guard pages · `switch_context` · creating a task · cooperative multitasking · preemption from the timer · sleeping, idling and exiting · `ps` and accounting · a multi-level feedback queue (optional) |
| **9** Synchronization | Interrupt control that nests · spinlocks · wait queues without lost wake-ups · semaphores, mutexes and condition variables · a keyboard that blocks · the classic problems as tests · audit the kernel and write the lock order |
| **10** User mode | Install a TSS · give every process an address space · the first jump into ring 3 · the system call gate · a user runtime · load programs as boot modules · the ELF loader · guard the door · write evil programs |
| **11** Process API | Processes, parents and the trap frame · frame reference counts · `fork`, the simple version · copy-on-write · `exit` and `waitpid` · `execve` with arguments · `brk` and a growing stack · minimal signals · `init` and the tests |

</details>

<details>
<summary><b>Stage IV: Persistence and userland (Phases 12–15)</b>. Disks, files, a C library, a shell, then hardening and the release.</summary>

| Phase | Tasks |
|---|---|
| **12** Storage | A disk for QEMU · find the drive with IDENTIFY · read and write sectors, polling · let the disk interrupt · a block device layer · partitions · a RAM disk · the buffer cache · tools and tests |
| **13** File systems | The VFS objects · descriptors and the basic calls · devices as files · a read-only root from a tar archive · path lookup · TurkFS, reading · TurkFS, writing · `stat`, `fstat` and `getdents` · tests |
| **14** Userland | Turk-libc: the skeleton · `malloc` in user space · `stdio` · pipes in the kernel · a real terminal · `tsh`, the Turk shell · core utilities · boot into the shell · test the userland |
| **15** Release | Fuzz the system call interface · audit the boundary · harden the kernel itself · users and permissions · benchmarks and one real optimisation · write the documentation · release v1.0 · defend your design · then choose a track beyond 100 % |

</details>

---

## How a phase document is organised

Every phase document has the same sections in the same order, so you always know where to look.

| Section | What you find there |
|---|---|
| **Title banner and progress bar** | Phase number, title, one-line goal, and the overall progress this phase covers (in red) |
| **At a glance** | Duration, difficulty (1–5 stars), what you need before starting, and what you will have at the end |
| **Why this phase** | Where the phase fits and what problem it solves |
| **Concepts you need** | The ideas to understand *before* writing code, with diagrams. Blue *concept* boxes hold key definitions |
| **Reading plan** | Book chapters with printed page numbers, each marked REQUIRED, RECOMMENDED or OPTIONAL, and why to read it |
| **Real-system lens** | How MINIX 3 does the same thing, with pointers into the printed source |
| **Build it** | Numbered tasks (Task 3.4 is phase 3, task 4) with starter code. Each ends with a green **Verify** line: the task isn't finished until that check passes |
| **Testing and debugging** | A table of symptom → likely cause → fix for the bugs this phase usually produces |
| **Milestone** | The exit criteria as a checklist. The phase is done when every box is ticked |
| **Self-check questions** | Answer them without looking. If you can't, go back to the reading |
| **Going further** | Stretch goals for when you finish early |
| **Books used** | The legend of book codes at the end of every document |

Coloured boxes appear throughout:

| Box | Meaning |
|---|---|
| Concept (blue) | A definition or idea you must understand |
| Hint (green) | A tip, a shortcut or a way out when something fails |
| Watch out (amber) | A common trap, or a command that can do damage |
| Real-system lens (MINIX) | How MINIX 3 solves the same problem |
| Decision (navy) | A design choice and the reasoning behind it |
| Exit criteria (red) | The milestone checklist |
| Self-check questions | Questions to test your understanding |

---

## Reading the reading plans

Each reading plan row looks like this:

| Priority | Book | Chapters | Pages | Why |
|---|---|---|---|---|
| REQUIRED | OSTEP | Ch. 1, Ch. 2 *Introduction to Operating Systems* | 1–22 | The three big ideas; the map for the whole plan |

- **REQUIRED**: read before starting the tasks.
- **RECOMMENDED**: makes the tasks easier and is worth the time.
- **OPTIONAL**: background and depth for later.

The usual order within a phase is OSTEP first, then the other REQUIRED items, then the practical chapter of LOB, then
the MINIX lens.

| Code | Book | Used for |
|---|---|---|
| **LOB** | Helin and Renberg, *The little book about OS development* (2015) | The practical build order (in `Books/book.pdf`) |
| **OSTEP** | Arpaci-Dusseau, *Operating Systems: Three Easy Pieces* (v0.90) | Main theory text |
| **OSC** | Silberschatz, Galvin and Gagne, *Operating System Concepts* (10th ed., 2018) | Reference and second explanation |
| **MOS** | Tanenbaum and Bos, *Modern Operating Systems* (4th ed., 2015) | Hardware, I/O, security, OS design |
| **OSDI** | Tanenbaum and Woodhull, *Operating Systems: Design and Implementation* (3rd ed., 2006) | MINIX 3 with its full source listing |

Two conventions matter when you look things up:

- **Page numbers are printed page numbers** from each book's table of contents, not PDF page indices. A PDF viewer's
  page box usually shows a different number.
- **MINIX source references** such as `kernel/proc.c (07400)` give the file and its starting line number in OSDI
  Appendix B.

---

## Working documents

The plan asks you to keep these files in `docs/`, next to the plan itself. Create each one when its phase asks for it.

| File | Created in | Contents |
|---|---|---|
| [`journal.md`](journal.md) | Phase 0 | Lab journal: date, what you tried, what happened, what you learned; a written hypothesis before every attempt on a long bug |
| `syscalls.md` | Phase 10 | System call reference: number, arguments, return value and errors for each call |
| Boundary audit table | Phase 15 | Every system call argument and how it is validated |
| Benchmark results | Phase 15 | The benchmark table, with one optimisation measured before and after |
| Architecture document | Phase 15 | Overview diagram, each subsystem, memory maps, system call table, lock order, TurkFS format, how to build, run, test and debug, known limitations (LaTeX next to the plan, or Markdown) |
| Design reflection | Phase 15 | Three to five pages comparing each Turk-OS subsystem with MINIX 3 and Linux |

The lock order itself lives in code, in a comment block in `include/locks.h` (Phase 9).

---

## Building the PDFs

The PDFs in [`pdf/`](pdf/) are committed, so you only need to build them after changing a `.tex` file.

### Requirements

`latexmk` and a TeX Live installation with pdfLaTeX. The install command for each platform (Arch, Fedora, Ubuntu/Debian,
macOS, Windows through WSL2) is in [setup.md, section 8](setup.md#8-latex-for-the-study-plan-optional). The
simplest choices are:

| Platform | Command |
|---|---|
| Arch | `sudo pacman -S --needed texlive-basic texlive-latex texlive-latexrecommended texlive-latexextra texlive-fontsrecommended texlive-fontsextra texlive-pictures texlive-binextra` |
| Fedora | `sudo dnf install latexmk texlive-scheme-full` |
| Ubuntu / Debian / WSL2 | `sudo apt install latexmk texlive-latex-extra texlive-fonts-extra texlive-fonts-recommended texlive-pictures` |
| macOS | `brew install --cask mactex-no-gui` |

On Fedora you can also start from a smaller scheme and install LaTeX packages by the file they provide:

```sh
sudo dnf install latexmk texlive-scheme-basic \
    'tex(XCharter.sty)' 'tex(zi4.sty)' 'tex(fontawesome5.sty)' 'tex(tcolorbox.sty)' \
    'tex(xltabular.sty)' 'tex(titlesec.sty)' 'tex(needspace.sty)' 'tex(enumitem.sty)'
```

If a build stops with `File 'something.sty' not found`, install the package that provides it: `'tex(something.sty)'`
on Fedora, or search [packages.ubuntu.com](https://packages.ubuntu.com) by file name on Ubuntu and Debian.

The style loads: `geometry`, `fontenc`, `inputenc`, `XCharter`, `helvet`, `zi4` (Inconsolata), `microtype`, `parskip`,
`xcolor`, `graphicx`, `array`, `booktabs`, `tabularx`, `xltabular`, `multirow`, `float`, `enumitem`, `amsmath`,
`amssymb`, `tcolorbox` (with `most`), `listings`, `tikz`, `fontawesome5`, `titlesec`, `fancyhdr`, `etoolbox`,
`xspace`, `caption`, `needspace`, `hyperref` and `bookmark`.

> **VS Code as a Flatpak (Linux):** its integrated terminal runs in a sandbox that can't see host programs such as
> `latexmk`. Build from a normal terminal, or use the host terminal profile from
> [setup.md](setup.md#vs-code-installed-as-a-flatpak-linux).

### Commands

Run these inside `docs/` (or from the repository root with `make -C docs ...`):

| Command | Result |
|---|---|
| `make` | Builds the roadmap and every phase document into `pdf/` |
| `make pdf/phase-03-gdt-idt-exceptions.pdf` | Builds one document |
| `make clean` | Deletes `build/` (auxiliary files) |
| `make distclean` | Deletes `build/` and `pdf/` |

`make` only rebuilds a PDF when its `.tex` file or `common/turkos.sty` has changed. Auxiliary files (`.aux`, `.log`,
`.toc`, ...) go to `build/`, which Git ignores; the finished PDFs go to `pdf/`, which Git tracks.

Each document runs `latexmk -pdf -interaction=nonstopmode -halt-on-error -outdir=pdf -auxdir=build`. The
[`.latexmkrc`](.latexmkrc) adds `common/` to `TEXINPUTS`, so every document finds `turkos.sty` no matter which folder
its source is in.

---

## Editing or adding a document

A new phase document only needs to be saved as `phases/phase-NN-short-name.tex`; the Makefile picks up every `.tex`
file in `phases/` automatically. Start from this skeleton:

```latex
\documentclass[11pt,a4paper]{article}
\usepackage{turkos}

\begin{document}
% number, title, one-line goal, overall progress from, to (percent)
\phaseheader{16}{Title}{One-line goal of the phase}{100}{100}

% duration, difficulty 1-5, prerequisites, outcome
\ataglance{2 weeks}{3}{What you need first}{What you will have at the end}

\section{Why this phase}
\section{Concepts you need}
\section{Reading plan}
\begin{readingplan}
\must & \OSTEP & Ch.~1 \emph{Title} & 1--10 & Why read it. \\
\end{readingplan}

\section{Build it}
\task{First task}
Explain the step.
\begin{lst}{c}{kernel/example.c}
int example(void) { return 0; }
\end{lst}
\verify{How to check that it works.}

\section{Testing and debugging}
\begin{pitfalls}
Symptom & Likely cause & Fix. \\
\end{pitfalls}

\section{Milestone}
\begin{milestonebox}[Name of the milestone]
\begin{checklist}
  \item First exit criterion.
\end{checklist}
\end{milestonebox}

\begin{questionbox}
\begin{enumerate}
  \item A self-check question.
\end{enumerate}
\end{questionbox}

\section{Going further}
\booklegend
\end{document}
```

The commands and environments `common/turkos.sty` provides:

| Name | Use |
|---|---|
| `\phaseheader{n}{title}{goal}{from}{to}` | Title banner with the overall progress bar |
| `\ataglance{duration}{difficulty}{prerequisites}{outcome}` | The *at a glance* box; difficulty is 1–5 stars |
| `\task{title}`, `\verify{text}` | A numbered build task and its check |
| `\begin{lst}{language}{caption}` | A code listing with a file name or caption (`c`, `asm`, `make`, `sh`, `plain`) |
| `\code{text}` | Inline code |
| `readingplan`, `\must`, `\should`, `\could` | The reading plan table and its REQUIRED / RECOMMENDED / OPTIONAL markers |
| `\LOB`, `\OSTEP`, `\OSC`, `\MOS`, `\OSDI` | Coloured book chips |
| `pitfalls` | The symptom / cause / fix table |
| `conceptbox{title}`, `tipbox[title]`, `warnbox[title]`, `decisionbox{title}`, `minixbox[title]` | Concept, hint, warning, decision and MINIX boxes |
| `milestonebox[title]`, `checklist`, `questionbox` | Exit criteria, tick-box list and self-check questions |
| `\tkth{text}` | A table header cell in the house style |
| `\tkprogress{from}{to}` | A stand-alone progress bar |
| `\nextphase{n}{title}`, `\booklegend` | The "next phase" link and the book legend at the end |

Colours are defined once in the style: `tkNavy`, `tkRed`, `tkBlue`, `tkGreen`, `tkAmber`, `tkGray`, the backgrounds
`tkLight` and `tkCode`, and one colour per book (`bkLOB`, `bkOSTEP`, `bkOSC`, `bkMOS`, `bkOSDI`). Use these names instead of new colours so every document looks
the same.
