# Static Recompilation & Source Ports
Static recomp **translates machine code to C/C++ automatically**, without hand-written source. It's faster to get a native port than with matching decomp, but the output isn't readable source.

- **N64Recomp**: "statically recompile N64 binaries into C code that can be compiled for any platform." Inspired by the IDO static recompilation used for matching decomp; cites jamulator (NES) as prior art. Powers Zelda 64: Recompiled. https://github.com/N64Recomp/N64Recomp , https://github.com/Zelda64Recomp/Zelda64Recomp
- **XenonRecomp**: converts Xbox 360 (PowerPC) executables to C++; x86-only output for now (uses x86 intrinsics); needs Clang 18+. Used by Unleashed Recompiled (Sonic Unleashed). https://github.com/hedge-dev/XenonRecomp , https://github.com/hedge-dev/UnleashedRecomp
- **Ship of Harkinian** (HarbourMasters): a PC port of OoT built **on the matching decomp** (not recomp), and you need your own ROM. Same group did 2 Ship 2 Harkinian (MM). https://github.com/HarbourMasters/Shipwright
- Related: PS2Recomp efforts, and the SM64 PC port from the decomp.

Key point: decomp → readable source → real ports/mods; recomp → fast native ports, plus mod hooks.

## 2026-10 update (GBA)
- mstan/gbarecomp: a general ARM/Thumb to C++ static recompiler with per-game repos (Minish Cap, Emerald, FR/LG, R/S, Mega Man Zero, etc.). FR/LG uses an interpreter plus JIT fallback that it caches to disk. https://github.com/mstan/gbarecomp
- FFTA got two recomps, 2026-10-09..12 (gba-recomp in Rust; gbarecomp). Secondary source: gamingworld.uk.

## How a GBA static recompiler works (gbarecomp, plainly)

> **Tutor's note.** An emulator reads each instruction *while the game runs* and pretends to be the CPU.
> A static recompiler does that translation *ahead of time*: each ARM/Thumb instruction becomes
> a line of C++, and a normal C++ compiler builds it into a native program. The game logic runs at full native
> speed, while a runtime library still pretends to be the GBA's graphics, sound and timers.

The [gbarecomp README](https://github.com/mstan/gbarecomp) describes four stages:
1. **Analyse:** find the code, follow branches, track ARM↔Thumb switches, and resolve indirect jumps where possible.
2. **Emit:** turn each discovered function into C++.
3. **Build:** compile that C++ with the shared GBA hardware runtime (PPU, audio, DMA, timers, saves, input).
4. **Fall back:** code that couldn't be found ahead of time (e.g. code copied into RAM, as Golden Sun does with
   its map overlays, see [16](16-golden-sun.md)) runs on an interpreter and gets cached as native code.

It ships **no ROMs, BIOS or ROM-derived source**. Each build checks a specific cartridge revision that the player supplies.

### Decomp vs recomp, side by side
| | Matching decomp | Static recomp |
|---|---|---|
| Output | Readable C a human wrote | Generated C++ nobody would read |
| Correctness check | Byte-identical rebuild | Plays the same (tests, comparing against an emulator) |
| Effort | Years of function-by-function work | One framework, then per-game fixes |
| Good for | Understanding, mods, ports, documentation | Native ports, widescreen, netplay quickly |
| Legal surface | Repo has no ROM/assets; you supply the ROM | Same model: you supply the ROM |

They help each other: a recomp gives you a working native build to compare against, and a decomp's names and
structs make recomp mods readable.

### Common mistakes
* **Calling recomp "decompiled source."** It's a translation of machine code, so it doesn't contain the original's structure or names.
* **Forgetting RAM-installed code.** Overlays and IWRAM-copied routines can't be found by scanning the ROM alone.
* **Assuming a recomp makes a decomp pointless.** They answer different questions (see the table).
