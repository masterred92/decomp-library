# The Legend of Zelda decomps (focus: GBA)

Researched 2026-10-07. Progress figures come from [zelda.deco.mp](https://zelda.deco.mp).

| Game | Project | Status | Contribute or start fresh? |
|---|---|---|---|
| **The Minish Cap** (GBA, Capcom/Flagship) | [zeldaret/tmc](https://github.com/zeldaret/tmc) | **100% matching, 100% nonmatching** ([tracker](https://zelda.deco.mp/games/tmc), updated 2026-09-27). Targets USA, JP, EU and both demos. Assets are extracted from your own ROM. Uses agbcc | Code matching is done. What's left: documentation, shiftability, and the PC port fork [999sian/tmc](https://github.com/999sian/tmc) (experimental 0.1.4, SDL3). **Best learning reference for GBA decomp** |
| **Four Swords** (GBA, with ALttP) | none found under zeldaret | ZeldaRET says "TMC will aid with the decompilation of Four Swords" and that the engines are similar ([FAQ](https://zelda.deco.mp/games/tmc)) | **Best start-fresh candidate**: shared engine with a 100% matched sibling |
| **A Link to the Past & Four Swords** (GBA port) | none found | No public GBA decomp located in search (the SNES original has separate disassembly work, not covered here) | Possible, but less support than Four Swords |
| Ocarina of Time (N64/GC) | [zeldaret/oot](https://github.com/zeldaret/oot) | Many versions supported. Default `gc-eu-mq-dbg`. Uses mips_to_c (m2c) | Mature, so contribute docs and versions |
| Majora's Mask (N64) | [zeldaret/mm](https://github.com/zeldaret/mm) | Active, 80 contributors, US 1.0 | Contribute |

Notes
* ZeldaRET has a strict policy: contributors must not have looked at leaked source ([mm README](https://github.com/zeldaret/mm)).
* TMC FAQ: "No specialist decompiler for ARM exists", so people use Ghidra or Hex-Rays for a first guess. About 640 kB of the 16 MB ROM is C code. Some parts use 3D hitboxes, and debug strings and flag names are left over.
* Use for our toolkit: run `gbadt init` on your own TMC ROM and compare our heuristic function list with tmc's symbols. That's a free accuracy benchmark, kept private, never committed.

---

## Deep dive: how a *finished* GBA decomp looks (zeldaret/tmc)

*Tutor note: read this as a tour of a house that's already built. Everything we build later should look like this.*
Source: I read a shallow clone of [zeldaret/tmc](https://github.com/zeldaret/tmc) on 2026-10-07. No ROM was used, and the repo contains no game assets.

### 1. The promise: same bytes out
The repo lists five target ROMs, each with a SHA1 (`tmc.sha1`, `tmc_jp.sha1`, `tmc_eu.sha1`, two demos). You run `make` (or `make eu`, `make jp`...) and the build must produce a file whose SHA1 is *identical* to the real cartridge. That one check is the whole quality bar: if a C function compiles to even one different byte, the hash fails. One codebase covers all five versions using `#ifdef` on `GAME_VERSION`/`REVISION`/language defines (see `GBA.mk` `CPPFLAGS`).

### 2. The compiler: agbcc, used carefully
- `Toolchain.mk`: `CC1 := $(AGBCC_PATH)/bin/agbcc`. agbcc is pret's rebuild of the old GCC 2.95-era compiler Nintendo shipped in the GBA SDK. It's the same compiler family as the Pokémon GBA decomps, so their tricks carry over.
- Flags (`GBA.mk`): `-O2 -Wimplicit -Wparentheses -Werror -Wno-multichar -g3`. Some files add `-mthumb-interwork`, and **one file, `eeprom.c`, is built at `-O1`**. Lesson: a real game isn't built with one setting everywhere, because the original developers' library code came with its own flags. When a function won't match no matter what you try, check the flags before the C.
- Pipeline: C → `cpp` (preprocess) → `agbcc` makes `.s` → GNU `as` → `ld` with `linker.ld` → `objcopy` padded with `0xFF` to 16 MB.
- `include/global.h` has matching "hacks" like `BLOCK_CROSS_JUMP asm("")` and `asm_unified(...)`. These are tiny nudges that make the old compiler emit the exact same instruction order.

### 3. How the repo is organised
| Folder | What lives there | Lesson |
|---|---|---|
| `src/` (~619 `.c` files) | Game code, split by *kind of thing*: `enemy/` (~100 files), `npc/`, `object/`, `manager/`, `item/`, `playerItem/`, `projectile/`, `menu/`, plus core files (`main.c`, `entity.c`, `collision.c`, `physics.c`, `player.c`, `script.c`) | One file per enemy/object is how the original devs probably worked too |
| `include/` | Headers with **struct layouts annotated with offsets** (`/*0x0c*/ u8 action;`) | Offsets are the bridge between assembly (`ldrb r0,[r4,#0xc]`) and C (`this->action`) |
| `asm/` (only ~17 `.s` files left) | crt0 (startup), veneers, a few hand-written routines | When finished, almost nothing is left as asm |
| `data/`, `assets/` | Tables and graphics/sound **definitions**; real bytes are pulled from *your* ROM by `make extract_assets` | Ship the recipe, not the ingredients |
| `linker.ld` (~1,700 lines) | The exact order every `.o` goes into ROM, and fixed RAM addresses for globals (`gSave = 0x02002A40`...) | This file *is* the game's memory map, and the most valuable thing to borrow |
| `tools/` | C++ helpers: `asset_processor`, `gbagfx`, `preproc`, `scaninc`, `agb2mid`/`mid2agb` (music), `tmc_strings` | Extraction/rebuild tools are half the work |
| `progress.py`, `asmdiff.sh`, `format.sh`, Doxygen | Progress tracking, diffs, formatting, [docs site](https://zeldaret.github.io/tmc) | Infrastructure that keeps 80+ contributors consistent |

Leftover names like `code_08049CD4.c` and `gUnk_02000010` show that even at "100%", not everything has a real name yet. Matching and *understanding* are two separate jobs.

### 4. Engine architecture (what Capcom/Flagship built)
- **Task loop** (`src/main.c`): `AgbMain()` runs `while (TRUE)` and calls `sTaskHandlers[gMain.task]()`. The tasks are Title, File Select (or Demo), Game, Game Over, Staff Roll and Debug. That's a simple state machine at the top.
- **Entities** (`include/entity.h`): almost everything on screen is an `Entity`, a struct in a linked list (`prev`/`next`) with `kind`, `id`, `type`, **`action`/`subAction`** (an index into a table of function pointers, "usually used to index a function table"), timers, sprite settings, `parent`/`child` links and more.
  - Pattern you'll see in hundreds of files: `static void (*const sActions[])(Entity*) = { Init, Idle, Attack, ... }; sActions[this->action](this);`. Once you recognise this in assembly (a load from a table, then `bl _call_via_rX`), whole files become easy.
- **Managers** are invisible entities that run room logic (bridges, bombable walls...), and **scripts** (`script.c`) run cutscenes and NPC behaviour from bytecode.
- Sound uses Nintendo's standard **m4a/MusicPlayer2000** library (`src/gba/m4a.c`), which is shared with Pokémon decomps.
- *Unverified:* that Four Swords uses the same `Entity`/action-table design. It's very plausible (same studio, earlier engine) but I haven't checked a binary.

### 5. The PC port fork
[999sian/tmc](https://github.com/999sian/tmc), "Project Picori", is a native Linux/Windows/macOS port built on the decomp (SDL3, experimental 0.1.x per earlier research). This is the "Halo in a browser" idea from the video: once the C source matches, you can recompile it for a different machine by swapping the GBA hardware layer (screen, input, sound registers) for SDL. You still need your own ROM for the assets. *Unverified:* build status and how much of the game is playable, since the fork's README was mostly empty when fetched.

### 6. Lessons for us
1. Decide the **SHA1 targets** on day one and make `make compare` the gate (our toolkit already does this).
2. Copy the folder layout: `src/<kind>/<thing>.c`, offset-annotated headers, and a single linker script as the memory map.
3. Expect **per-file compiler flags**.
4. Keep assets out: extract from the user's ROM at build time.
5. Match first, name later. `sub_`/`gUnk_` names are fine for a long time.

---

## Four Swords (GBA): what's known
- Four Swords isn't a separate cartridge. It's the multiplayer half of **"A Link to the Past & Four Swords"** (2002). Flagship/Capcom made Four Swords and then Minish Cap (2004), which is why ZeldaRET says TMC "will aid" a Four Swords decomp.
- **Existing work:** [camthesaxman/zeldaalttp](https://github.com/camthesaxman/zeldaalttp) is a *disassembly* of the combined cart (mostly assembly, ~5% C, 5 contributors, last push Oct 2022). [gbadisasm](https://github.com/jiangzhengwenjz/gbadisasm) ships a config `alttpafs.cfg` for the US cart. Text: Four Swords strings are plain ASCII, while the ALttP half's text is encoded ([Zelda Legends](http://www.zeldalegends.net/index.php?p=128)). Its build target and compiler are written up just below (read 2026-10-07).
- So a "start fresh" Four Swords project should really **start from zeldaalttp's disassembly** rather than from zero, and should coordinate with ZeldaRET's Discord to avoid duplicating work.

### zeldaalttp: ROM target and compiler (read 2026-10-07)
Read from a shallow clone of [camthesaxman/zeldaalttp](https://github.com/camthesaxman/zeldaalttp)
at HEAD `d3802f5` (merge of PR #6, 2022-10-15). **There is no README**, so everything here comes
from the `Makefile`, `checksum.*`, `ldscript.txt` and `asm/rom_header.inc`. No ROM was used, so we
haven't run `make compare` ourselves.

**The ROM it rebuilds**
| Field | Value |
|---|---|
| Output | `zeldaalttp.gba`, checked by `make compare` (`sha1sum -c checksum.sha1`) |
| SHA1 | `a272055abbbf6c26b0cd54c87395d01699589161` (MD5 `3287ca66e5cc285a9fe3a922051e84c6`) |
| Header | title `GBAZELDA`, game code `AZLE`, maker `01`, version byte 0, header checksum `0x8A` |
| Edition | `AZLE` = the North American "A Link to the Past & Four Swords", first revision (version 0). Region read from the game code's last letter `E`; we haven't checked it against a dump database |
| Size | linked at `0x08000000`, then `objcopy --gap-fill 0xFF --pad-to 0x8800000`, so an 8 MiB image with `0xFF` filling the unused space |
| Needs | your own dump saved as `baserom.gba` in the repo root: the data files pull bytes straight from it (about 4,700 `.INCBIN "baserom.gba"` lines) |

**The compiler: agbcc** (the same family as Minish Cap)
* `CC1 := tools/agbcc/bin/agbcc`, flags `-mthumb-interwork -Wimplicit -Wparentheses -O2`. C goes
  through `arm-none-eabi-cpp` (`-Itools/agbcc/include -iquote include -nostdinc -undef`), then
  agbcc, then `.ALIGN 2, 0` is appended to each `.s` before assembling. This is the classic pret pipeline.
* **Per-file flag:** `-fprologue-bugfix` for `rom_0800D4F0`, `main`, `math`,
  `file_select`, `rom2` and `main_2`. There's no `src/main_2.c`, so that last line is a leftover. The
  flag is a pret agbcc option: in `thumb.c` it makes the compiler re-check whether a function really
  contains a far jump before deciding to save `lr`, which changes the prologue bytes. In other words,
  some of the game's files were built with a slightly different compiler revision. Same lesson as tmc:
  expect per-file compiler settings.
* `CC1_OLD := old_agbcc` is defined but no rule uses it yet (it would come in for the m4a sound library).
* Links agbcc's `libgcc.a` and `libc.a`. `tools/agbcc` is gitignored: you build pret/agbcc yourself
  (our `gba-decomp-toolkit/scripts/install_agbcc.sh <repo>` does that).
* Binutils come from **`$DEVKITARM/bin/arm-none-eabi-*`**, so `DEVKITARM` must be set. Pointing it at
  our ARM GNU toolchain folder should work, since it has the same `bin/` layout (**[untested]**).
* `make` also builds `tools/gbagfx` (PNG to GBA graphics converter) automatically.
* Small gotcha: the Makefile uses `$(SCANINC)` for header dependencies but never defines it, so
  dependency scanning silently does nothing. After editing a header, `make tidy` first.

**How it's split**
* One `ldscript.txt` lists every object in ROM order: `asm/crt0`, big chunks `asm/rom1`…`rom6`,
  and C files placed between them (`src/main.c`, `math.c`, `rom2.c`, `file_select.c`, `game_select.c`,
  `hud.c`, `interface.c`, `rom3a.c`, `rom_0800D4F0.c`).
* `src/lttp/` holds the ALttP sprite (enemy/NPC) code that's been matched: `sprite.c`, `sprite2.c`,
  `sprite_witch.c`, `sprite_faerie.c`, `sprite_bottle_vendor.c`, `sprite_heart_refill.c`,
  `sprite_pickup.c`, `sprite_unused.c`; `asm/lttp/sprite_bee.s` is still asm.
* `data/4swords_text.s` is the Four Swords text; RAM variables are named in `sym_ewram.txt` and `sym_iwram.txt`.
* **Progress, roughly:** about 6,700 lines of C against about 663,000 lines of asm and data, with
  about 7,430 function-start macros in `asm/`. That fits the "~5% C" figure above. Function macros come in
  two spellings (`THUMB_FUNC_START` ×7,249 and `thumb_func_start` ×180), a sign of different
  contributors' tools.
* Helpers: `calcrom.pl` (progress from the `.map` file), `asmdiff.sh <addr> <len>` (objdump diff of
  `baserom.gba` vs the build), `rename_sym.sh`.

**What it means for us**
* Our compiler guess for Four Swords was "agbcc `-O2`, like tmc". zeldaalttp's matched files are built
  exactly that way, which is good evidence, though we haven't run their `make compare`. Step 5 of the plan
  below (match 3 functions to confirm) is still worth doing.
* To use it with our toolkit: `gbadt signatures` can find tmc functions in this cart, then
  `import-symbols` names them. Because zeldaalttp isn't shiftable yet (raw `.INCBIN` + fixed
  addresses), keep any renames in our private scaffold or send them upstream as `rename_sym.sh` runs.

### How tmc can speed it up
- **Function names via signatures.** Functions shared between the games (m4a sound, agbcc's libgcc, memcpy/decompress helpers, maybe entity/collision core) will have the same or near-same bytes. Build tmc on *your own* ROM, hash each function's bytes (masking out branch/pointer targets), and look for those hashes in your own ALttP&FS ROM. Every hit gives you a name and a matching C body to try.
- **Our tool:** `gbadt signatures build/match` (added 2026-10-07) fingerprints functions with addresses masked and writes a symbol list; `gbadt import-symbols` applies it to a scaffold. Tested on our own demo ROM built in two layouts (5/5 functions found).
- **Headers:** borrow `entity.h`-style struct layouts as a *first guess*, then verify each offset against the Four Swords assembly. Structs from two years earlier will differ.
- **Compiler:** try agbcc `-O2` with tmc's flags first. *Unverified* until one Four Swords function matches.
- Legal: keep everything private on your PC. Contribute upstream through ZeldaRET (which has a no-leaked-source rule) instead of publishing a Nintendo-title decomp ourselves.

### Concrete plan (small steps, one commit each)
1. ~~Read zeldaalttp's README/Makefile and write down how it's split and which ROM SHA1 it targets.~~ Done 2026-10-07 (above; there is no README).
2. Privately: dump your own cart, run `gbadt init`, and check that `make compare` is OK.
3. ~~Toolkit feature: `gbadt signatures` hashes functions with masked relocations and outputs a symbol list. Test it on our demo ROM.~~ Done 2026-10-07.
4. Privately: build tmc from your own TMC ROM, export its `nm`, run signatures across both, then `import-symbols`. Record the hit count in the library (numbers only, no code).
5. Pick 3 tiny matched-by-signature functions and confirm they compile byte-identical with agbcc. That confirms the compiler.
6. Then ask in ZeldaRET's Discord whether a Four Swords effort exists and join or seed it.

---

## OoT / MM: how to contribute (brief)
- [zeldaret/oot](https://github.com/zeldaret/oot) is essentially fully matched. Useful work now is naming and documenting variables/functions, adding more ROM versions, and tidying code. MIPS (IDO compiler), so our m2c and MIPS tools apply.
- [zeldaret/mm](https://github.com/zeldaret/mm) still has decomp work left. The usual path is to join the Discord, read CONTRIBUTING, claim an actor (one `ovl_*` file) so nobody duplicates it, decompile it with m2c + asm-differ + decomp-permuter, then open a PR.
- Both require that you **haven't seen leaked source**. Never open the Nintendo "gigaleak" material if you want to contribute.

## Toolkit feature built for this note
`gba-decomp-toolkit` now has `python3 -m gbadt import-symbols <project> <symfile> [--offset N]`. It reads `nm`, linker-script or plain address lists, renames matching `sub_XXXXXXXX` functions (files, labels, config), reports the rest, and keeps the build byte-identical. It's tested only on our own homebrew demo ROM.

Update 2026-10-07: `gbadt signatures` now produces those symbol lists by masked-byte fingerprint (see `docs/signatures.md` in the toolkit).
