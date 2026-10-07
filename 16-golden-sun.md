# Golden Sun & Golden Sun: The Lost Age (GBA, Camelot)

Researched 2026-10-07 by reading each project's public README/docs (shallow clones, no ROMs).
Numbers are what the repos said that day. **[unverified]** marks things I couldn't confirm myself,
usually because confirming them needs a ROM we don't have.

> **Tutor's note.** A "matching decomp" means writing C that, when compiled with the *exact*
> compiler Camelot used, turns back into the *exact* bytes on the cartridge. The game itself
> is the answer key: if the bytes match, you got it right. Everything below is about how the
> Golden Sun community set that answer-checking machine up.

## TL;DR
* **GS1 is well underway.** [Coaltergeist/goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp)
  counts **3,747 of 5,794 original functions as matched C (≈64.7%)**, plus 23 "fakematches"
  (byte-identical but with suspicious C) and 2,024 still in assembly
  (from its `progress_snapshot.json`, HEAD `3ede4d1`, 2026-10-06 22:31 PT). The earlier 44% figure in this note is out of date.
* **The compiler is not agbcc.** Game code uses a **patched GCC 2.96 snapshot (dated 2000-07-31)**,
  rebuilt in [Coaltergeist/camelot-gcc](https://github.com/Coaltergeist/camelot-gcc). Nintendo's
  stock sound library inside the game still needs pret's `old_agbcc`.
* **The Lost Age (TLA)** uses the same engine but a *slightly modified* compiler (two Thumb
  backend changes, see below). Only [PascalPixel/alchemy](https://github.com/PascalPixel/alchemy)
  is publicly working on TLA right now.
* **Best way in for Kenny:** contribute to goldensun-decomp, starting with an existing
  "almost matching" candidate (see the last section).

## 1. The projects and how they relate
| Project | What it is | Notes |
|---|---|---|
| [gsret/goldensun](https://github.com/gsret/goldensun) | The original **disassembly** of GS1 (USA): every function turned into labelled assembly that rebuilds the ROM | "All known code disassembled and symbolized". Data not disassembled; overlays verified uncompressed only (no matching compressor). Not "shiftable" (moving anything breaks pointers). The foundation everyone else builds on |
| [Coaltergeist/goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp) | **Matching C decomp** built on gsret's disassembly | Active (commit 2026-10-06). Tracked on [decomp.dev](https://decomp.dev/Coaltergeist/goldensun-decomp). Accepts PRs |
| [Coaltergeist/camelot-gcc](https://github.com/Coaltergeist/camelot-gcc) | The recreated compilers (source + build/install scripts), pret/agbcc-style | gcc-2.96 (GS1 production), gcc-3.0 (TLA research baseline, not wired in), pruned `old_agbcc` |
| [PascalPixel/alchemy](https://github.com/PascalPixel/alchemy) | Separate **"clean-room"** AI-heavy decomp of **both games, all 6 languages each** (12 editions), Japanese as the source edition | Its own compiler build, **agscc** (also GCC 2.96 2000-07-31). Rule L2: it never looks at other GS projects; rule L3: no leaked material. Closed to outside contributions until both games are done |
| [GoldenSunHacking](https://github.com/GoldenSunHacking) org | Older community codebase: `Psynergy` ("modding tools"), `gsmagic`, `GS1-TBS-map`, `GS2-TLA-map`, `GS3-DD-map`, `GoldenSunCompression` (fork) | Mostly 2023, little README detail **[unverified contents]** |
| [FutureFractal/GS-headers](https://github.com/FutureFractal/GS-headers) | Reverse-engineered C headers for the GBA games | Credited by goldensun-decomp for names and Ghidra annotations |
| [SBird1337/SunAnalyzer](https://github.com/SBird1337/SunAnalyzer) | C# analyser for map-code files | Small, MIT |
| Atrius' TLA editor | Item/enemy/text editor. Mirror (GPL-3.0, GameMaker 8.1 + C++ DLL): [nikitalita/GoldenSunTLA_Editor](https://github.com/nikitalita/GoldenSunTLA_Editor); [forum guide](http://forum.goldensunhacking.net/index.php?topic=1.0) | Its table offsets are a ready source of struct layouts for TLA **[unverified: haven't cross-checked offsets]** |

**Two-camp warning.** Alchemy deliberately never reads goldensun-decomp, and goldensun-decomp
has its own provenance rules. If Kenny contributes to one, don't paste work across to the
other. Pick one per function and keep notes on where ideas came from.

## 2. The compiler: patched GCC 2.96
*Why it matters:* the same C gives different bytes on different compilers. Get this wrong and
nothing ever matches.

* **What it is:** a GCC 2.96 *development snapshot* from 2000-07-31 (not a release). Community
  work (credited to Tarpman and Karathan) pinned it down from habits in the binary.
* **Key flag: `-fcall-used-r4`.** Normally a function must save register r4 before using it. With
  this flag r4 becomes scratch, so functions start saving at r5. Alchemy reports 265 of 266 GS1
  functions that save `lr` start saving at r5. That fingerprint is how they knew. (This also
  explains why Nintendo's stock sound library, built without the flag, needs a different compiler.)
* **What camelot-gcc patches** (from its README):
  - host fixes so 2000-era GCC builds on modern Linux/macOS (`config.guess`, `-std=gnu17`, `-fcommon`);
  - **determinism:** the optimizer hashed symbols by memory address, so output could vary run to
    run; patched to hash by name. This *can* change generated code;
  - `.align N, 0` so padding between functions is zero bytes like the original.
* **Production flags** (goldensun-decomp `Makefile`, 2026-10-07):
  `-O2 -mthumb -mthumb-interwork -mcpu=arm7tdmi -fno-builtin -nostdinc -ffreestanding -fcall-used-r4 -fno-strict-aliasing`
  The build runs `xgcc -S` to make assembly, then modern `arm-none-eabi-as`. A couple of files have documented exceptions (e.g. one
  battle-animation file uses `-fstrict-aliasing`).
* **Other compilers in the same ROM:** `old_agbcc` for the m4a ("Sappy") sound engine and most
  Flash-save library C; 51–53 ARM functions are hand-written assembly (kept as asm).

### How to get it (no binaries are shipped by anyone; you build it)
```sh
sudo apt install build-essential binutils-arm-none-eabi python3 python3-venv git less
git clone https://github.com/Coaltergeist/goldensun-decomp.git
git clone https://github.com/Coaltergeist/camelot-gcc.git
cd camelot-gcc
./build.sh gcc296 && ./install.sh ../goldensun-decomp gcc296   # -> tools/gcc296/
./build.sh agbcc  && ./install.sh ../goldensun-decomp agbcc    # -> tools/agbcc/  (builds -j1)
```
Our toolkit can use the same install: `gbadt init ... --compiler gcc-2.96-patched` with
`GCC296_DIR=/path/to/tools/gcc296` (see `gba-decomp-toolkit/docs/compilers.md`).
**Built and checked on our box (2026-10-07)**, camelot-gcc `1197a54`, user space under `~/tools/gcc296`,
no sudo, under a minute on 8 cores. camelot-gcc's ROM-free smoke corpus passed 3/3: its exact assembly
equals what the compiler that rebuilt the full GS1 ROM produced. Our own toy C showed the fingerprints:
`push {r5, ...}` (r4 never saved), `adds rX, rY, #0` register copies, and `x * 10` as shift-adds.
One gotcha: `build.sh` stops with `host compiler missing: g++` if there's no `g++`, even though gcc296
compiles no C++. `CXX=clang++ ./build.sh gcc296` gets past it. One command does it all:
`gba-decomp-toolkit/scripts/install_camelot_gcc.sh` (steps in its `docs/compilers.md`). Still needs a
ROM: building goldensun-decomp itself and `make compare`.

## 3. Build setup and verification (goldensun-decomp)
1. Put your own USA ROM at `baserom.gba`; `sha1sum` must be `5c4695205413df7db52b9a184815a07783999971`.
2. `make -j1 clean && make -j1 compare` → must print `goldensun.gba: OK` and pass **all 96 overlay**
   comparisons. Use serial (`-j1`) builds.
3. Optional diff viewer: asm-differ pinned at `0dd09af`, in its own venv under `tools/asm-differ`.
4. **Before editing anything:** `python3 tools/create_diff_baseline.py` saves the "known good"
   objects to `expected/`.
5. Compare one function: `bash ./run-diff.sh -mo FUNCTION --no-pager --format plain`.
   For overlay functions, select the map: `GOLDENSUN_DIFF_MAP=overlays/rom_XXXXXX/overlay.map`.
Ubuntu/WSL2 is the supported host. ROMs, extracted assets and build output must never be committed.

## 4. Repository layout
| Path | Contents |
|---|---|
| `src/` | C by subsystem: `battle/`, `battle_anim/` (82 move animations in `moves/`), `field/` (+ `moves/` = field Psynergy), `rpg/` (party, items, Djinn, summons), `ui/`, `decompress/`, `math/`, `sound.c`, `save.c`, `task.c`, `main.c`… |
| `src/maps/` | **One file per code overlay** (each town/dungeon has its own code), plus shared `common/` |
| `src/lib/` | Nintendo libraries (m4a sound, Flash save) |
| `src/non_matching/` | Unfinished C "candidates", mirroring `asm/`, with `INDEX.md` |
| `asm/` | Functions not yet in C; each pulled into its C file with `INCLUDE_ASM("asm/…/Func.s")` |
| `overlays/` | 96 overlay linker scripts (`rom_779188/` etc.) |
| `include/` | Headers (`global.h`, `task.h`, `rpg.h`, `message.h`, `file_table.h`, `libcamelot.h`…) |
| `*.sym` | Address maps: `wram.sym` (RAM variables), `event.sym`, `message.sym`, `file_table.sym` |
| `fakematch.txt`, `unmatchable.txt`, `candidate_scores.json`, `progress_snapshot.json` | Bookkeeping |

## 5. How contributors pick and match functions
The project's rules ([CONTRIBUTING.md](https://github.com/Coaltergeist/goldensun-decomp/blob/main/CONTRIBUTING.md)) in plain words:
* **Byte-identical isn't enough.** The C must plausibly be what Camelot wrote. Types, signedness
  and access sizes have to be justified from the code and its callers.
* **No new fakematches.** Banned tricks: pinning variables to registers, empty `asm("")` barriers,
  dummy reads, odd temporaries just to steer the compiler, hiding these in macros.
* **Use production flags only.** Per-file experiments don't count.
* **Workflow:** baseline → replace one `INCLUDE_ASM` with C → compare the *whole* object (neighbours,
  symbols, relocations) → `tools/finalize_progress.py --objdiff …` (verifies ROM + 96 overlays,
  refreshes reports) → `tools/check_repository.py` and unit tests → PR describing behaviour,
  reasoning, compiler, validation.
* **Picking work:** the easy on-ramp is `src/non_matching/` candidates with a high
  `fuzzy_match_percent` in `candidate_scores.json`. Compare with
  `python3 tools/compare_candidate.py src/non_matching/<TU>/<Func>.c` (EXACT / DIFF / ERROR).
  Other routes: small leaf functions in `asm/`, or cleaning entries in `fakematch.txt`.
* Tools they credit: decomp.me, asm-differ, decomp-permuter (config `permuter_settings.toml`), objdiff.

## 6. Engine architecture (what the folders tell us)
*Partly inferred from file and header names; behaviour details **[unverified]** until read in a ROM-backed build.*
* **Task system** (`task.h`, `task.c`): a table of 20 tasks, each a function pointer + priority +
  status, run every frame. That's the main loop's "scheduler".
* **Field** (`src/field/`): map loading, actors and their movement, cutscenes, weather, Djinn on
  the map, transitions. **Field Psynergy** has one file per spell (`moves/move.c`, `growth.c`,
  `frost.c`, `douse.c`, `force.c`, `halt.c`, `catch.c`, `carry.c`…) driven from `field/psynergy.c`.
* **Per-map code overlays:** each area's script-like logic is real compiled C in its own overlay,
  stored compressed in ROM and decompressed into RAM when you enter. That's why there are 96
  overlays and why gsret couldn't put them back without a matching compressor. Map `*_GetEvents`
  functions check the "current event" field against IDs in `event.sym`; that's the "scripting":
  C, not a bytecode language **[inference]**.
* **Battle** (`src/battle/`): `battle.c`, `mechanics.c` (damage/effects), `enemy.c`, plus a
  `debug_battle_test.c` (a leftover debug feature). **Battle animations** (`src/battle_anim/`)
  have one file per Psynergy/summon effect (82 of them), plus shared camera/cast/palette helpers.
* **RPG data** (`src/rpg/`): party, items, moves (`HasMove`), Djinn, summons, PCs, RNG.
* **Assets:** loaded by index through `GetFile(n)` from a global file table (`file_table.h`).
* **Compression** (`src/decompress/`): several LZ variants (`lz.c`, `lz1.s`, `lz2.s`, `lz16.s`,
  `sprite_lz.c`), tilemap unpacking, and a Huffman decoder (`huffman.s`).
* **Text:** Huffman where each character's tree depends on the *previous* character (context
  trees, starting from context 0). Tools and both games' scripts:
  [romhack/GoldenSunCompression gs-huffman](https://github.com/romhack/GoldenSunCompression/tree/master/gs-huffman);
  walkthrough of finding the decoder: [tutorial](https://github.com/romhack/GoldenSunCompression/blob/master/golden%20sun%20tutorial%20in%20progress.txt).
  Message IDs are named in `message.sym`; printing lives in `ui/` (`AdvanceMsgText`, `PrintBattleText`).
* **Hot code in IWRAM:** some routines are copied to fast RAM at boot (`fixup_ram_code.s`); a decomp
  must link them at their RAM addresses.

## 7. How The Lost Age shares the engine
* Same engine, a year later, same compiler *family*. Alchemy's `games/COMMON/` exists to share
  code between the two, and its stated goal is "sharing as much code between the games as possible".
* **The difference is the compiler** (Alchemy AGENTS.md, rule K2; this is their claim, **[unverified]** by us):
  - `-mthumb-split-constants`: build awkward constants from a shifted byte + add instead of loading from
    ROM. 13,907 of 14,101 constant sequences in TLA follow it; GS1 has 18.
  - `-mthumb-call-via-lr`: indirect calls go through `lr` instead of `_call_via_rX` stubs (TLA: 2,549 vs
    3 stubs; GS1: 596 stubs). Implies TLA game code dropped ARM interworking.
  - These are not in any public GCC 2000–2002, so they're reconstructed Camelot changes.
* camelot-gcc keeps vanilla gcc-3.0 as a TLA *starting point* but says it can't reproduce TLA natively.
* **Practical upshot:** a function matched in GS1 often has a twin in TLA. Same C, but it compiles
  differently because of those two options. A TLA scaffold built on GS1's names would get a big head
  start (our toolkit's `import-symbols` was built for exactly this).

## 8. Who made it (developer-profile angle)
Camelot Software Planning (Takamura brothers); published by Nintendo. Code habits visible in the
decomp: a house `libcamelot.h` (e.g. a `CAMELOT_MEMCLEAR` macro calling through a function
pointer), heavy use of overlays, real C for map scripts, `-fcall-used-r4`. Programmer credits
**[unverified]**, see 15-developer-profiling.md.

## 9. Legal reminder
Both decomp projects ship no ROM and require your own. Golden Sun is published by Nintendo, so
takedown risk exists. Keep Kenny's GS work in a private fork until we decide otherwise.

## 10. Suggested first matching task (once Kenny has his own ROM)
**`HasMove`** (`asm/rpg/move/HasMove.s`, ROM `0x08078BC0`, about 20 instructions). It's small, the
name says what it does (does this character know move X?), and there's already a parked candidate
`src/non_matching/rpg/move/HasMove.c` scoring **81.8%** (48 bytes). Steps:
1. Build camelot-gcc (done on the box, see section 2) + goldensun-decomp, check `make compare` passes, then run `create_diff_baseline.py`.
2. `python3 tools/compare_candidate.py src/non_matching/rpg/move/HasMove.c`, read the diff.
3. Usual suspects for the last ~20%: `u8` vs `int` loop counters, `s16` vs `u16` move IDs, loop shape
   (`for` vs `while`), and the struct field's access width.
4. Once EXACT, move it into `src/rpg/move.c` in place of the `INCLUDE_ASM`, finalize, open a PR.
Backup: `Func_80167ac` in `ui/ui` (67.1%, 44 bytes).
