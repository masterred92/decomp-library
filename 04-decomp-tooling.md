# Matching-Decomp Tooling
| Tool | Purpose | Link |
|---|---|---|
| decomp.me | Collaborative web "scratches": paste asm, write C, live diff vs. target with real compilers | https://decomp.me/ , https://github.com/decompme/decomp.me |
| m2c (formerly mips_to_c) | Asm → C draft decompiler; MIPS, PowerPC, ARM support | https://github.com/matt-kempster/m2c |
| asm-differ | Side-by-side diff of target vs. your compiled asm (`diff.py`) | https://github.com/simonlindholm/asm-differ |
| splat | Splits a ROM/binary into asm/data/asset segments from a YAML config (N64, PSX, PS2, PSP) | https://github.com/ethteck/splat |
| objdiff | GUI/CLI object-file diffing; standard on GC/Wii projects; feeds decomp.dev progress | https://github.com/encounter/objdiff |
| decomp-permuter | Randomly permutes C to search for a matching codegen | https://github.com/simonlindholm/decomp-permuter |
| decomp-toolkit (dtk) | GC/Wii DOL/REL splitting and project setup | https://github.com/encounter/decomp-toolkit |
| objdiff ARM support | objdiff's README lists ARM (GBA, DS, 3DS) alongside ARM64, MIPS, PowerPC, SuperH and x86 | https://github.com/encounter/objdiff |
| gbadisasm | GBA disassembler used by pret-style projects to produce the starting `.s` files | https://github.com/jiangzhengwenjz/gbadisasm |
| luvdis | Another GBA (Thumb) disassembler, aimed at producing assembler-ready output | https://github.com/aarant/luvdis |
| coddog | Finds similar or duplicate functions across binaries, useful for sibling games like GS1 vs The Lost Age | https://github.com/ethteck/coddog |

## How the tools fit together (plain version)

> **Tutor's note.** Think of a matching loop as **draft → compile → compare → nudge**. Each tool
> owns one step:
> * **Splitting** (splat, decomp-toolkit, gbadisasm plus a hand-written linker script on GBA) cuts the
>   binary into one assembly file per function, so you can swap one of them for C.
> * **Drafting** (m2c, Ghidra, an LLM) gives you a rough first C version.
> * **Comparing** (asm-differ's `diff.py`, objdiff, decomp.me) shows your output next to the target, instruction by instruction.
> * **Nudging** is you, or the permuter, rewording the C until the diff is empty.
> * The **referee** is the project's full `make compare`, which checks the whole ROM hash.

### decomp-permuter, properly
The [README](https://github.com/simonlindholm/decomp-permuter/blob/main/README.md) says it supports MIPS (IDO, possibly GCC),
PowerPC and **ARM32**. You give it a C file and the target `.s` (`./import.py file.c file.s`), and it scores random
rewrites by how close the output gets. You can also point it at specific ideas with macros like
`PERM_GENERAL(a, b)` (try both versions), `PERM_LINESWAP(...)` (try line orders) and `PERM_RANDOMIZE(...)` (limit the
random changes to one region).

**Its big limit:** it scores *bytes*, not *meaning*. On Golden Sun's `Func_8015e8c`, its best result after
5 minutes compiled closer but **did something different** from the original, so we rejected it
([25 §3](25-golden-sun-matching-lessons.md)). Always reread a permuter result as if a stranger wrote it.

### asm-differ vs objdiff
* **asm-differ** (`diff.py`): runs in the terminal, rebuilds when files change and is configured per project. It's common on N64/GBA.
* **objdiff**: GUI and CLI, compares object files directly, and powers [decomp.dev](https://decomp.dev/) progress.
  It started out standard on GC/Wii and now lists ARM.
* Both mark a register-only difference differently from a *real* instruction difference. Learning to tell them apart
  is most of the skill (see "What didn't match" in [25](25-golden-sun-matching-lessons.md)).

### Common mistakes
* **Diffing with the wrong compiler or flags.** A near-match with the wrong flags can never become exact. Golden Sun
  needs camelot gcc 2.96 with `-fcall-used-r4` ([16](16-golden-sun.md)).
* **Trusting a function-level match alone.** Section padding and alignment can still break the ROM, so always run the full compare
  ([25 §1](25-golden-sun-matching-lessons.md)).
* **Letting the permuter add nonsense.** Dummy variables and odd casts that it finds are fakematches.
* **Leaving rodata inside a function's `.s`.** Split the data out first (e.g. `GetSpriteVoiceEntry` in [25](25-golden-sun-matching-lessons.md)).

See also: [06](06-platform-notes.md) GBA/Thumb, [07](07-workflow.md) workflow, [09](09-box-tool-availability.md) what's installed here.
