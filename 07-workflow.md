# Practical Workflow: Decompiling a Game
1. **Pick a target you legally own.** Small and well-documented is best (start on decomp.me scratches or join an existing project).
2. **Identify the platform, CPU, executable format, and compiler** (strings, toolchain signatures, SDK libs). The compiler version decides whether matching is possible.
3. **Dump your own copy** (ROM/disc/exe) and verify its hash against the project's expected SHA1.
4. **Load into Ghidra/IDA**, get the memory map right, and apply SDK signatures (libultra, Nintendo SDK, MSVC CRT) to name library code.
5. **Split** the binary into segments (splat for MIPS, decomp-toolkit for GC/Wii) and build a "non-matching but rebuilding" baseline from asm that reproduces the identical ROM.
6. **Pick small leaf functions first.** Generate a draft with m2c or Ghidra (or an LLM), then iterate in decomp.me with asm-differ/objdiff until it matches. Use the permuter for stubborn register-allocation diffs.
7. **Recover types/structs**, name things, and document them. Extract assets with tools like ZAPD/splat so the repo ships **no copyrighted assets**.
8. **Track progress** (decomp.dev / objdiff reports) and keep CI building a matching ROM.
9. **Downstream**: once it matches, the source is moddable/portable (e.g. SM64 PC port, Ship of Harkinian). Alternatively, static recomp (N64Recomp) for faster native ports.
10. **Non-matching RE** (PC/modern games): Ghidra + debugger (x64dbg), dynamic analysis, and hooking for mods. Skip byte matching here.

Learning: decomp.me tutorials, https://github.com/zeldaret/oot/blob/main/docs/decompiling_tutorial.md , pret wiki.

## A worked example from our own PRs (added 2026-10-14)

> **Tutor's note.** This walks through one real function, `Func_8012038` from
> [PR #34](https://github.com/Coaltergeist/goldensun-decomp/pull/34), step by step. The C shown is ours, from the public PR.
> It isn't the original game's code.

1. **Pick it from the worklist.** It's small (about 27 instructions, one branch, no calls), in `map_data.c`, with nobody else
   working on it. We rank functions by `instructions + 3×branches + 2×calls` ([25 §5](25-golden-sun-matching-lessons.md)).
2. **Read what it does.** It reads byte 2 of a 4-byte map tile at `(x >> 20, z >> 20)` in a 128-wide map for
   layer `n & 3`, falling back to a buffer if the field state is null.
3. **Describe the data as structs first** ([25 §2.1](25-golden-sun-matching-lessons.md)):
   ```c
   struct MapTile { unsigned char unk0, unk1, unk2, unk3; };
   struct MapLayer { struct MapTile *tiles; unsigned char pad4[0x2c]; };
   struct MapLayerState { unsigned char pad0[0x130]; struct MapLayer layers[4]; };
   ```
4. **First draft** (`return tiles[x + (z << 7)].unk2;`): it compiles, but the diff shows the pointer maths in a different order.
5. **Nudge it.** Split the access into two steps, in the order the original seems to use:
   ```c
   tiles += x + (z << 7);
   return tiles->unk2;
   ```
   Now the diff is empty ([25 §2.3](25-golden-sun-matching-lessons.md)).
6. **Referee:** `make clean && make compare` reports the ROM and all 96 overlays OK.
7. **Ship small.** Put it in a PR with two related functions and explain any struct choices in the PR body.

## The loop, as a checklist (reference)
| Step | Tool | Done when |
|---|---|---|
| Choose | Worklist script, upstream activity | Small, nobody else working on it, no rodata in its `.s` |
| Draft | m2c / Ghidra / LLM | It compiles |
| Compare | asm-differ / objdiff / decomp.me ([04](04-decomp-tooling.md)) | Only register or order differences remain |
| Nudge | You (permuter last) | The diff is empty and the C is *plausible* |
| Verify | `make compare` | The whole ROM hash matches |
| Integrate | Merge with your other open branches and rebuild | Still matches |

## Common mistakes
* **Starting with big functions.** Branches and calls multiply the ways things can go wrong.
* **Skipping the full build** because the function diff is clean.
* **Big PRs.** Maintainers review small ones faster. We keep a few open at a time.
* **Committing generated files** (progress reports) when the project generates them itself.
* **Committing the ROM or extracted assets.** Never do this (see [08](08-legal-ethics.md)).
