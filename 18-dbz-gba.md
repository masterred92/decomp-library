# Dragon Ball Z on GBA

Researched 2026-10-07. We have **no ROMs** and downloaded none. Everything below comes from public repos, wikis and tooling docs.
Labels: **[checked]** = I read it in the source repo myself. **[unverified]** = inferred or second-hand, confirm before relying on it.

## The games and who made them
| Game | Year | Developer | Publisher |
|---|---|---|---|
| The Legacy of Goku (LoG I) | 2002 | Webfoot Technologies | Infogrames |
| The Legacy of Goku II (LoG II) | 2003 | Webfoot | Atari (ex-Infogrames) |
| Buu's Fury | 2004 | Webfoot | Atari |
| Supersonic Warriors | 2004 | Arc System Works + Cavia | Banpresto / Atari |

Sources: [Wikipedia, LoG series](https://en.wikipedia.org/wiki/Dragon_Ball_Z:_The_Legacy_of_Goku_(series)), [Wikipedia, Supersonic Warriors](https://en.wikipedia.org/wiki/Dragon_Ball_Z:_Supersonic_Warriors), [TCRF](https://tcrf.net/Dragon_Ball_Z:_Supersonic_Warriors).

---

## 1. The Buu's Fury disassembly ([2genkidev/buusfury](https://github.com/2genkidev/buusfury))

**What it is, in plain terms:** a project that rebuilds the exact same ROM (SHA1 `f1c4b07554d2a3b1ad2f325307051e775ce68087`, USA) from files in the repo plus your own copy of the game. It is at a very **early stage**. [checked]

What is actually in it [checked]:
* `build.bat`: a Windows-only script. It runs ARM's tools in order: `armcpp` (C++ compiler), `armasm` (assembler), `armlink` with a **scatter file** (`scatter.ld`, ARM's version of a linker script that says "put this in ROM, put that in EWRAM/IWRAM"), then `fromelf -bin` to turn the linked file into a raw `.gba`. Finally it checks the SHA1.
* `asm/`: about 400 lines of hand-written assembly: the ROM header, `crt0.s` (startup code that runs before `main`), RAM layout, and `rest_of_the_game.s`, which is just a placeholder.
* `src/strings.cpp` (about 3,000 lines of game text as data) and `src/color_transforms.cpp` (256-entry colour lookup tables for screen effects). These are **data written as C++**, not decompiled functions yet.
* `assets/`: splash, intro and dialog-box images turned back into BMPs, rebuilt with `grit` and recompressed with a custom JCALG1 tool in `tools/compress`.
* **The key trick:** `armasm` can't do `INCBIN file, offset, size` the way GNU `as` can, so `tools/incbin.bat` cuts slices out of `baserom.gba` into separate files. Almost all of the 8 MB ROM is still copied in as raw slices (e.g. `0x071c90-0x3be3d4`, `0x3c33f8-0x7b89a8`).

**Lesson for Kenny:** this is the classic first step of any decomp. You start with "the whole ROM as one blob", prove you can rebuild it byte-for-byte, then carve out one piece at a time (header, then strings, then images) and keep the checksum green after every carve. Code functions come last.

Practical blockers: it needs ADS 1.2 (see section 5), it's Windows-only, it has one contributor, and the last push was 2025-07 (from earlier research, [unverified] today).

## 2. The Buu's Fury static recomp ([mstan/DragonBallZBuusFuryRecomp](https://github.com/mstan/DragonBallZBuusFuryRecomp))

**What a static recomp is:** instead of rewriting the game in C by hand, a tool reads the ARM/Thumb machine code and **translates every instruction into C++ automatically**, then compiles that into a normal Windows program. The original GBA hardware (graphics, sound, timers) gets emulated around it. You get a native PC game, but the generated code is unreadable, with no real names or structure. That's the same family as N64Recomp and, likely, the "Halo in a browser" style ports. [checked]

Details [checked]:
* Built on mstan's reusable **[gbarecomp](https://github.com/mstan/gbarecomp)** framework. The DBZ game is described as its "proving ground", and there's a write-up called [Recomp + AI: 5 Months Later](https://1379.tech/recomp-ai-5-months-later/). The repo also ships a `CLAUDE.md`, so it was built with AI agents.
* `game.toml` is the per-game config: load address `0x08000000`, 8 MB, entry point, the code range to scan (`0x080000C0-0x08030000`), the ROM's SHA1/MD5, EEPROM save type (8 KB), and **real BIOS required** (`hle = false`).
* v0.0.1 preview. It boots through attract mode into gameplay, with "strict-static validation" (meaning no instruction fell back to an interpreter in the parts tested). It isn't tested through the whole game.
* There's an optional **Adaptive Widescreen** mod (up to 480×160) that streams more map chunks and widens actor visibility, not just stretching the picture.
* You bring your own ROM and BIOS. Nothing copyrighted is in the repo.

**Why it matters for us:** a recomp doesn't care which compiler Webfoot used, so it **sidesteps the ADS problem completely**. It's also the closest DBZ thing to what the video showed.

## 3. DragonByteZ ([spicybung/DragonByteZ](https://github.com/spicybung/DragonByteZ))

MIT-licensed C++ analyser/extractor (GUI and CLI), created 2026-07 [checked]. It supports LoG I EU (`ALGP`), LoG II EU/US (`ALFP`/`ALFE`) and Buu's Fury US (`BG3E`). Commands: `header`, `analyze`, `graphics`, `soundtrack`, `decompress <offset>`.
* `compression.cpp` (~200 lines) has a bit reader that pulls **32-bit words MSB-first**, with **Elias-gamma** length codes and copy-from-earlier-output matches. That's the shape of the JCALG1/aPLib family of LZ compressors. [checked code; the "JCALG1" label comes from other tools]
* `log1_runtime.cpp`, `gsf_player.cpp` (uses `viogsf`, a GBA sound emulator) and `sprite_analysis.cpp` mean it plays the soundtrack by running the game's own code, and finds sprites itself.
* The fact that one tool covers all three Webfoot games with one decompressor is good evidence of a **shared engine**.

## 4. Compression research
* **[Arefu/Legacy](https://github.com/Arefu/Legacy)** plus its [wiki](https://github.com/Arefu/Legacy/wiki): "they compress the ever living hell out of basically everything." Its `DBZKit` C# editor suite (`DrGero`, `Bulma`, `Dragon Radar`) has a `JCALG1.cs` decoder used by sprites, tilesets, dialog and boot data. [checked]
* **[luatsenpai gist](https://gist.github.com/luatsenpai/b3e48b093100e553f61d8824cf64f273)**: JCALG1 text decompress, repack and repoint for Buu's Fury. [from earlier research]
* buusfury's `tools/compress` bundles a JCALG1 library to *re*-compress assets so they match exactly. [checked]
* **What JCALG1 is:** a small LZ77-style compressor by Jeremy Collake, popular on late-90s PCs for packing .exe files. It's **not** one of the GBA BIOS's built-in LZ77/Huffman routines. That's a studio choice: Webfoot brought a PC tool to the GBA. Better compression than the BIOS LZ77, at the cost of CPU time. [the JCALG1 history is general knowledge, unverified for here]
* **Caution:** the Arefu repo also contains an IDA database (`.i64`) built from a LoG II ROM, plus save files. An IDA database can hold the game's bytes, so **we haven't opened or used it**, and we shouldn't copy it. The Markdown notes are fine to read.

## 5. Webfoot's engine: what's shared across LoG I, LoG II and Buu's Fury
Based on DBZKit's `Engine-Notes.md`, `BytecodeVM_OpCodes.md` and `Audio-Notes.md` (from LoG II US, read in IDA 2026-09):
* **Script bytecode VM** [checked notes]: a stack machine with 3 dispatch tables. Main ops `0x00-0x1D` (push, jump, step), about 146 game actions (`OpCodes.csv`: dialogue, move NPC, play audio…), and a variable get/set table. Opcode `0x11` ends a script. NPCs, quests and cutscenes are all data plus scripts, not hard-coded C.
* **Map loading** [checked notes]: `Map_LoadInternal` loads gates, then items, objects, triggers, music and `mapScripts[]`. Each record starts with a handler pointer (create NPC / actor / enemy). Entities use **vtables** (tables of function pointers per object type), which points to **C++ or C-with-objects**, matching buusfury's use of `armcpp`.
* **Behavior lists** for NPC AI: small "wait / face / wander" behaviours cycled forever.
* **Custom sound engine** [checked notes]: *not* Nintendo's usual m4a "Sappy" driver. It's Webfoot's own software mixer: 8-bit samples at about 16 kHz, Timer0 + DMA1/2 FIFOs, 15 music and 16 SFX voices, tracker-style songs, a shared 101-instrument bank.
* **JCALG1 for everything**, and one DragonByteZ decompressor for all three games (see above).
* **Shared across all three games?** Strongly suggested by DBZKit and DragonByteZ supporting all three with the same formats. Exact code reuse (same functions, same addresses) is **[unverified]**. LoG I is the oldest and probably the most different.

**Tutor note:** the sound engine is a "studio fingerprint". A team that wrote their own mixer, used a PC compressor and ARM's own compiler was a PC/embedded-style studio, not a Nintendo-SDK-by-the-book team. That fits the developer-profiling angle.

## 6. ADS 1.2 (armcc) vs gcc/agbcc, and what it means for matching
**Background:** most GBA decomps (pret's Pokémon, Minish Cap) use **agbcc**, a rebuilt copy of the old GCC 2.95-based compiler Nintendo's SDK shipped. Because it's GCC and its source is public, anyone can rebuild it and match. Webfoot instead used **ARM Developer Suite 1.2** (2001-2002): `armcc` (ARM C), `tcc` (Thumb C), `armcpp`/`tcpp` (C++), `armasm`, `armlink`. [buusfury README checked; build.bat uses `armcpp`]

Typical differences you'd see in the assembly (general, **[unverified]** for this game specifically):
* **Different register allocation and instruction scheduling.** The same C gives different but equivalent instructions, so a gcc guess will *never* byte-match.
* **Literal pools** (constants stored after a function) are placed differently, and armcc likes to share them between functions.
* **Function calls:** armcc uses ARM's own veneers / interworking stubs (`__call_via_rX`-style vs gcc's `_call_via_rX`), with different names and layout.
* **Helpers:** division and 64-bit maths call ARM's `__rt_*` / `__aeabi`-era library routines, not libgcc's `__divsi3`.
* **C++:** armcpp has its own name mangling and vtable layout, so object code looks different from g++'s.
* **Object format:** ADS produces ELF/DWARF2 but links with scatter files and `fromelf`, so a GNU `ld` linker script doesn't translate one-to-one.
* Fun fact: ARM's compiler was the one ARM itself sold. It often makes *tighter* code than old GCC, which is why some studios paid for it.

**Can it be obtained legally?**
* ADS 1.2 was **commercial and licence-locked (FLEXlm)**. ARM discontinued it (replaced by RVDS, then the Keil MDK / Arm Compiler 5/6). There's **no free legal download** that I know of. Copies you find online are usually pirated. [unverified that no legacy licence route exists]
* Legal-ish routes: an old boxed or academic copy with a licence, or asking Arm. Arm sometimes provides legacy compilers to licensees. **Arm Compiler 5** (armcc 5.x, via Keil MDK; there's a free size-limited edition) is a descendant and might come *close*, but it won't produce identical code to 1.2. [unverified]
* **Don't** download a cracked ADS for us. It's against our "keep it clean" rule.

**Alternatives if we can't get ADS 1.2:**
1. **Recomp route** (mstan's gbarecomp): no compiler match needed.
2. **Non-matching ("functional") decomp:** write C that behaves the same, checked by running it rather than by bytes. That's easier but loses the automatic "MATCH" check that makes AI decomp so fast.
3. **Hybrid:** keep functions as assembly (the buusfury approach) and only rewrite in C the ones we can test. Our toolkit now has this: `gbadt init ... --compiler ads12` makes an asm-only project (rebuilds the exact ROM, `match` disabled) or, with `--mode notes-only`, just a function table to document (added 2026-10-07).
4. **Documentation route:** like Arefu, name functions, structs and formats in Ghidra and publish *notes*, no code. Lowest legal risk, and good for learning.

## 7. Supersonic Warriors (Arc System Works + Cavia)
* A 2D versus fighter, so a **totally different engine** from Webfoot's RPGs. Arc System Works (Guilty Gear) brings fighting-game frame data, hitboxes and state machines.
* TCRF documents a debug menu (code `03000891:0C`, English version) and leftover content. [from TCRF, not re-checked today]
* **No disassembly, decomp or recomp found** in our searches [unverified: absence is hard to prove]. The compiler is unknown. A Japanese studio is more likely to have used Nintendo's SDK toolchain (agbcc-style GCC), which *would* be good news for matching. **[unverified guess]**
* There's a sequel, Supersonic Warriors 2 (DS, 2005), out of scope.

## 8. Contribute vs start fresh
| Target | Best route | Why |
|---|---|---|
| Buu's Fury | Help buusfury carve out data, or study the recomp | Groundwork exists. Code matching is blocked on ADS 1.2 |
| LoG II | Documentation (Arefu wiki style) with our own Ghidra notes | Best-researched engine, and scripts/maps are well mapped |
| LoG I | Later, once LoG II is understood | Oldest, probably the most different |
| Supersonic Warriors | Check the compiler first; if it's agbcc, it's the best *matching* DBZ target | Open field, maybe a friendlier compiler |

## Recommended next step
Write an `armcc`-free lesson: build a tiny C function with **gcc** for ARM/Thumb (`arm-none-eabi-gcc`, which we can legally install), then hand-write what armcc-style output tends to look like (literal pool placement, call veneers), and compare. The toolkit half is done: the `ads12` profile (2026-10-07) runs ADS projects in asm-only or notes-only mode. The lesson half needs no ROM.
