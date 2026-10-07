# 24: Linux, Linus Torvalds and Git

Linux is the operating system almost every decomp project is built on, and Git is the tool every one of them is stored in. Both were started by the same person, Linus Torvalds. This note covers the history, how kernel development actually works, Linus's own repos, and the best places to learn Linux and operating systems. Throughout, it points out where each piece connects to decompilation.

Checked 2026-10-07: every GitHub repo here was confirmed to exist with `gh api repos/OWNER/NAME`. Dates come from widely documented history. Verify specifics before you quote them.

## Study these 5 first

| # | Resource | Why first |
|---|---|---|
| 1 | [jlevy/the-art-of-command-line](https://github.com/jlevy/the-art-of-command-line) | One page of the shell skills that every decomp tool assumes you have. |
| 2 | [0xAX/linux-insides](https://github.com/0xAX/linux-insides) ([book site](https://0xax.dev/books/linux-inside/)) | Walks through the real kernel from the moment the power comes on, assembly included. This is reading low-level code with a guide. |
| 3 | [mit-pdos/xv6-riscv](https://github.com/mit-pdos/xv6-riscv) | A whole Unix-like OS in about 10,000 lines of C, used in MIT's OS course. Small enough to read completely, unlike Linux. |
| 4 | [mewmew/dissection](https://github.com/mewmew/dissection) and [corkami/pics](https://github.com/corkami/pics) | Take one ELF binary apart byte by byte. ELF is the format our GBA builds pass through before `objcopy` turns them into a `.gba` file. |
| 5 | [git/git](https://github.com/git/git), starting with [the first commit](https://github.com/git/git/commit/e83c5163316f89bfbde7d9ab23ca2e25604af290) | Linus's first version of Git was tiny (about 1,000 lines of C). It's a great small C codebase to read, and it's the tool we use every day. |

## 1. The story of Linux

- **1991, the post.** On 25 August 1991, Linus Torvalds, then a 21-year-old student at the University of Helsinki, posted to the Usenet group `comp.os.minix`: *"I'm doing a (free) operating system (just a hobby, won't be big and professional like gnu) for 386(486) AT clones."* MINIX was Andrew Tanenbaum's teaching OS. Linus used it, disliked its limits, and wrote his own kernel for his new 386 PC.
- **Version 0.01** came out in September 1991. It was a kernel only. The tools around it (shell, compiler, utilities) came from the **GNU project**.
- **The GPL (1992).** Linux's first licence banned commercial use. With version 0.12 (early 1992) Linus moved it to the **GNU General Public License v2**. The GPL means anyone may use, change and share the code, but anything they distribute must stay open under the same terms. Linus later called this one of the best decisions he made. Companies could contribute without anyone being able to close the code, and that is a big part of why Linux won.
- **1.0** came out in March 1994. Linux now runs most servers, every Android phone, most supercomputers, and the build machines of nearly every decomp project.
- **Tanenbaum–Torvalds debate (1992).** Tanenbaum posted "LINUX is obsolete", arguing that a microkernel design (like MINIX) was better than Linux's monolithic kernel. The thread is a famous, readable argument about OS design trade-offs.
- **The book: *Just for Fun: The Story of an Accidental Revolutionary*** (2001), by Linus Torvalds and David Diamond. It's an easy, funny memoir of his childhood, the first kernel, and his "it's fun" philosophy of open source.
- **GNU/Linux.** Richard Stallman started GNU in 1983 to build a free Unix. By 1991 GNU had a compiler, shell and tools but no finished kernel. Linux filled that gap, which is why some people say "GNU/Linux".

## 2. How Git began (2005)

- From 2002 the kernel was managed with **BitKeeper**, a proprietary version-control system whose owner, BitMover, let free-software developers use it at no cost.
- In early 2005, Andrew Tridgell (of Samba) wrote a tool that talked to BitKeeper servers by **reverse engineering its network protocol**. BitMover withdrew the free licence. *A reverse-engineering dispute is literally why Git exists.*
- Linus started writing Git in April 2005. His first commit, "Initial revision of "git", the information manager from hell", is dated **7 April 2005** (15:13 PT). Git was hosting its own development within days and the kernel within weeks.
- In **July 2005** he handed maintenance to **Junio Hamano**, who still maintains it today. Git 1.0 came out in December 2005.
- **Key ideas:** every file version is stored as an object named by a hash of its contents. A commit is a snapshot plus its parent commits. Every copy of a repo is a full copy of the history (distributed). It's fast and very hard to corrupt silently.
- **[git/git](https://github.com/git/git)** is a *publish-only mirror*. Real development happens by emailing patches to the Git mailing list, and GitGitGadget converts GitHub pull requests into emails. `Documentation/SubmittingPatches` explains the process.

**Decomp link:** every matching decomp lives in Git. Branches, `git bisect` (finding the commit that broke a match) and reading history (`git log -p`) are daily tools. GitHub itself was built on top of Git in 2008.

## 3. How kernel development works

The kernel does **not** use GitHub pull requests. [torvalds/linux](https://github.com/torvalds/linux) is a read-only mirror of Linus's tree on kernel.org.

- **Mailing lists.** Changes are sent as plain-text **patches** by email (`git format-patch` then `git send-email`) to the right subsystem list and to the LKML (Linux Kernel Mailing List). Everything is archived and searchable at **lore.kernel.org**. The `b4` tool fetches and applies patch series from there.
- **Maintainers.** The `MAINTAINERS` file (almost 1 MB) lists who owns each part of the code. `scripts/get_maintainer.pl` tells you who to email for a given change. `scripts/checkpatch.pl` checks coding style before you send.
- **The hierarchy.** Contributor → subsystem maintainer (reviews and collects patches in their own Git tree) → Linus (pulls whole maintainer trees). It's a "network of trust": Linus mostly reviews maintainers, not individual patches.
- **Release cycle.** After each release there is a **2-week merge window** where new features go in. Then come about 7 weekly release candidates (`-rc1`, `-rc2`, …) with fixes only. A new version ships roughly every 9–10 weeks. **linux-next** tests what's coming next, and **stable/LTS** kernels (led by Greg Kroah-Hartman) get bug-fix backports.
- **Signed-off-by.** Each patch carries a `Signed-off-by:` line certifying the Developer Certificate of Origin (DCO), which says you have the right to submit the code. This came in after the SCO lawsuits of 2003–04. It's a good model for showing where code came from, which matters in decomp too (see [08-legal-ethics.md](08-legal-ethics.md)).
- **Rules worth stealing:** "don't break userspace" (never break existing programs), small self-contained patches with a clear "why" in the message, and public review of everything.
- **Recent changes:** Rust became a second language for kernel code from 6.1 (December 2022). In 2018 Linus took a short break and the kernel adopted a Code of Conduct.

Start with `Documentation/process/submitting-patches.rst` in the kernel tree, and [gregkh/kernel-tutorial](https://github.com/gregkh/kernel-tutorial) for a walkthrough of a first kernel patch.

## 4. Linus Torvalds's own GitHub repos

Profile: [github.com/torvalds](https://github.com/torvalds). These are his own projects, not forks. They show how he works on small things, not only the kernel.

| Repo | What it is | Why look |
|---|---|---|
| [torvalds/linux](https://github.com/torvalds/linux) | Mirror of the mainline kernel tree (one of the most-starred repos on GitHub) | The real thing. Browse `arch/arm/` (the ARM CPUs that GBA, DS and Switch use) and `arch/x86/entry/syscalls/syscall_64.tbl`, the list of every x86-64 syscall number. |
| [torvalds/uemacs](https://github.com/torvalds/uemacs) | His personal version of the tiny MicroEMACS text editor | Small, old-school C. Shows the editor he has used for decades. |
| [torvalds/test-tlb](https://github.com/torvalds/test-tlb) | "Stupid memory latency and TLB tester" | Measuring hardware behaviour (caches, TLB) with a tiny C program. |
| [torvalds/GuitarPedal](https://github.com/torvalds/GuitarPedal) | "Linus learns analog circuits" | Him learning a new field from scratch in public, the same way we learn. |
| [torvalds/AudioNoise](https://github.com/torvalds/AudioNoise) | Random digital audio effects | Digital signal processing (DSP) in C. |
| [torvalds/HunspellColorize](https://github.com/torvalds/HunspellColorize) | Wraps `less` to highlight spelling mistakes | Classic Unix style: glue small tools together. |
| [torvalds/pesconvert](https://github.com/torvalds/pesconvert) | Converter for Brother PES embroidery-machine files | **Reverse engineering a binary file format**, by Linus himself, for his wife's sewing machine. |
| [torvalds/ScrollWheel](https://github.com/torvalds/ScrollWheel), [torvalds/1590A](https://github.com/torvalds/1590A) | RP2350 microcontroller scroll-wheel toy, and a guitar-pedal enclosure design (OpenSCAD) | Hardware hobby projects. |

His forks (`libgit2`, `subsurface-for-dirk`, `libdc-for-dirk`) exist for syncing with collaborators. **Subsurface**, the dive-log app he co-created, really lives at `Subsurface-divelog/subsurface`.

## 5. The GNU/Linux toolchain, and why decomp depends on it

| Tool | Origin | What it does for decomp |
|---|---|---|
| **GCC** (GNU Compiler Collection) | Richard Stallman, first release 1987. Mirror: [gcc-mirror/gcc](https://github.com/gcc-mirror/gcc) | The compiler behind most GBA games. **agbcc** (pret) is a fork of old GCC, and Golden Sun's camelot-gcc is a patched gcc-2.96 ([16-golden-sun.md](16-golden-sun.md)). Matching decomp means reproducing what one exact old GCC version produced. |
| **GNU binutils**: `as`, `ld`, `objdump`, `readelf`, `nm`, `objcopy`, `strings` | GNU project, from about 1990. Official repo at sourceware.org (mirror: [gnutools/binutils-gdb](https://github.com/gnutools/binutils-gdb)) | `objdump -d` disassembles the code. `readelf` and `nm` list sections and symbols. `objcopy -O binary` turns the linked ELF into the final `.gba` ROM image. asm-differ wraps `objdump` ([04-decomp-tooling.md](04-decomp-tooling.md)). |
| **GDB** (GNU Debugger) | Stallman, 1986. Lives in the same binutils-gdb tree | Step through code instruction by instruction. mGBA and other emulators provide a GDB stub (a link a debugger can attach to), so you can debug a running GBA game. [pwndbg/pwndbg](https://github.com/pwndbg/pwndbg) makes it much friendlier. |
| **ELF** (Executable and Linkable Format) | Unix System V Release 4 (around 1989). Linux switched to it from `a.out` in the mid-1990s | The container for compiled code: headers, sections (`.text` for code, `.data`, `.bss`), symbols and relocations. Every decomp build makes `.o` ELF files, links them into one ELF, then extracts the raw ROM. Ghidra and objdiff read ELF. |
| **Syscalls** | Unix design, numbered per CPU architecture | How a program asks the kernel for things (open a file, write). [strace/strace](https://github.com/strace/strace) shows every syscall a program makes. Recognising syscall stubs helps when you decompile PC or Linux binaries. Consoles like the GBA have no OS; they use BIOS calls (`swi`) instead, which is a similar idea ([06-platform-notes.md](06-platform-notes.md)). |
| **make** | Unix (1976), with GNU make widely used | Every pret-style decomp builds with a Makefile. |

**Linux as the decomp workstation OS:** nearly every matching decomp (pret, the N64 projects, Golden Sun) documents Linux (or WSL on Windows) as the main build environment. Old compilers like agbcc and IDO recomp, the shell scripts, and Python tooling all assume a Unix-like system. The box we use is Linux ([09-box-tool-availability.md](09-box-tool-availability.md)).

## 6. Learning repos: Linux, the kernel and systems

**Command line and everyday Linux**
- [jlevy/the-art-of-command-line](https://github.com/jlevy/the-art-of-command-line): one page, in many languages.
- *The Linux Command Line* by William Shotts: a free, complete book at [linuxcommand.org/tlcl.php](https://linuxcommand.org/tlcl.php) (it isn't hosted on GitHub). It's the best start-to-finish shell book.
- [inputsh/awesome-linux](https://github.com/inputsh/awesome-linux): curated Linux projects and resources. (`luong-komorebi/Awesome-Linux-Software` is also useful but archived.)
- [trimstray/the-book-of-secret-knowledge](https://github.com/trimstray/the-book-of-secret-knowledge): a huge list of command-line tools, one-liners and cheat sheets.

**Build a Linux system yourself**
- **Linux From Scratch** ([linuxfromscratch.org](https://www.linuxfromscratch.org/), read-only mirror [lfs-book/lfs](https://github.com/lfs-book/lfs)): build a working Linux system from source code, one package at a time. You end up understanding the toolchain (binutils → GCC → glibc), the same chain decomp compilers are built with.
- [buildroot/buildroot](https://github.com/buildroot/buildroot): builds small embedded Linux systems automatically.

**The kernel**
- [0xAX/linux-insides](https://github.com/0xAX/linux-insides): boot process, interrupts, memory, syscalls.
- [sysprog21/lkmpg](https://github.com/sysprog21/lkmpg): *The Linux Kernel Module Programming Guide*, updated for modern kernels.
- [linux-kernel-labs/linux-kernel-labs.github.io](https://github.com/linux-kernel-labs/linux-kernel-labs.github.io): university kernel course with labs.
- [gregkh/kernel-tutorial](https://github.com/gregkh/kernel-tutorial): how to write and submit a kernel patch.

**Operating systems in general**
- [mit-pdos/xv6-riscv](https://github.com/mit-pdos/xv6-riscv) (current) and [mit-pdos/xv6-public](https://github.com/mit-pdos/xv6-public) (older x86 version): MIT's teaching Unix. The commentary book is free on the [6.1810 course site](https://pdos.csail.mit.edu/6.1810).
- *Operating Systems: Three Easy Pieces* (free at ostep.org) with [remzi-arpacidusseau/ostep-projects](https://github.com/remzi-arpacidusseau/ostep-projects): the friendliest OS textbook, with matching projects.
- [cfenollosa/os-tutorial](https://github.com/cfenollosa/os-tutorial): build a tiny OS from a boot sector upward, step by step.
- [tuhdo/os01](https://github.com/tuhdo/os01): a book on writing an OS from scratch, including reading Intel manuals and ELF.
- [littleosbook/littleosbook](https://github.com/littleosbook/littleosbook): *The little book about OS development*, short and practical.
- [s-matyukevich/raspberry-pi-os](https://github.com/s-matyukevich/raspberry-pi-os): learn by writing an OS for the Raspberry Pi (ARM), modelled on Linux. ARM is the GBA's CPU family.
- Community wiki: [wiki.osdev.org](https://wiki.osdev.org), the reference for hobby OS developers.

**Binaries, debugging and tracing (closest to decomp)**
- [mewmew/dissection](https://github.com/mewmew/dissection): a "hello world" ELF taken apart byte by byte.
- [corkami/pics](https://github.com/corkami/pics): one-page posters of ELF, PE and other file formats.
- [strace/strace](https://github.com/strace/strace): trace the syscalls a program makes.
- [pwndbg/pwndbg](https://github.com/pwndbg/pwndbg): GDB plugin for reverse engineers (alternatives: [hugsy/gef](https://github.com/hugsy/gef), [cyrus-and/gdb-dashboard](https://github.com/cyrus-and/gdb-dashboard)).
- [brendangregg/perf-tools](https://github.com/brendangregg/perf-tools): see what the kernel and programs are doing while they run.
- [lief-project/LIEF](https://github.com/lief-project/LIEF): read and modify ELF, PE and Mach-O executable files from Python or C++.

## Why this matters for us

1. **Our tools are GNU/Linux tools.** GCC, binutils, GDB, make and ELF are the decomp pipeline. Knowing where they came from explains why old compiler versions behave the way they do.
2. **Git came out of a reverse-engineering fight.** It's a reminder that RE has real legal and social consequences ([08-legal-ethics.md](08-legal-ethics.md)).
3. **The kernel's process is a model for a big collaborative codebase:** maintainers, small reviewed patches, clear commit messages, sign-offs. Large decomp projects (OoT, pret) work in a similar way.
4. **Reading an OS teaches you to read any big C program:** boot code, interrupts and memory maps, the same ideas as a GBA game's startup code and IRQ handlers.

See also: [21-learning-paths.md](21-learning-paths.md) (the OS and C stages), [22-language-history.md](22-language-history.md) (why C won), [06-platform-notes.md](06-platform-notes.md), [04-decomp-tooling.md](04-decomp-tooling.md).
