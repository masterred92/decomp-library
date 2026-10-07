# Golden Sun & Golden Sun: The Lost Age (GBA, Camelot)

Researched 2026-10-07. Figures are as the project READMEs state them on that date.

## TL;DR
* **Golden Sun 1 already has a matching decomp in progress.** Best move: contribute, don't start fresh.
* **The compiler is not agbcc.** Community work points to a **patched gcc-2.96** (an early GCC-3.0-family snapshot). Our toolkit's default agbcc assumption is wrong for this game.
* **The Lost Age has no public matching decomp that I found.** The engine is shared, so GS1 work carries over. This is the realistic "start fresh" target later.

## Existing projects
| Project | What it is | Status |
|---|---|---|
| [gsret/goldensun](https://github.com/gsret/goldensun) | Disassembly of GS1 (USA). Builds `goldensun.gba` SHA1 `5c4695205413df7db52b9a184815a07783999971` with arm-none-eabi binutils | "All known code disassembled and symbolized". Data not yet disassembled. Overlays (map code) are built and verified uncompressed only, because there's no matching compressor yet. Not shiftable |
| [Coaltergeist/goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp) | Matching C decomp built on gsret's disassembly | Byte-identical at HEAD. **2,539 / 5,745 Thumb functions matched (44.2%)**. The 51 ARM functions are hand-written asm. 97 overlay banks and 16 main-ROM banks. Compiler: patched gcc-2.96 via a separate `camelot-gcc` repo, with gcc-3.0 for cross-checks and pret's `old_agbcc` only for the stock m4a ("Sappy") audio engine. TLA is described as a later target |
| [PascalPixel/alchemy](https://github.com/PascalPixel/alchemy) | "Automated, clean-room" decomp aiming at byte-perfect English GS1, then TLA and other languages | Active. Claims zero use of leaked material |
| [GoldenSunHacking (GitHub org)](https://github.com/GoldenSunHacking) | Community codebase: `GS1-TBS-map`, `GS2-TLA-map`, `GS3-DD-map`, `gsmagic`, `GoldenSunCompression` | Repos have little README detail |
| [SBird1337/SunAnalyzer](https://github.com/SBird1337/SunAnalyzer) | C# static analyser for the Camelot engine's map-code files (Capstone) | Small, MIT |

## Formats and tools
* **Text**: modified Huffman where each character's tree depends on the previous character (context trees, starting from context 0). Tool and both games' scripts: [romhack/GoldenSunCompression gs-huffman](https://github.com/romhack/GoldenSunCompression/tree/master/gs-huffman). Walkthrough of finding the decoder, LZ graphics and code copied into IWRAM: [tutorial](https://github.com/romhack/GoldenSunCompression/blob/master/golden%20sun%20tutorial%20in%20progress.txt).
* **Code in IWRAM / overlays**: hot routines and per-map code are copied or decompressed into RAM at runtime (tutorial above, and gsret's overlay notes). A decomp has to model these as separate link units at their RAM addresses.
* **Atrius' editor (TLA)**: item, enemy, enemy-group and text viewers ([forum guide](http://forum.goldensunhacking.net/index.php?topic=1.0), [preview thread](https://www.goldensun-syndicate.net/forum/topic/13005-gs-tla-editor-preview-warning-lots-of-pictures/)). Source mirror (GPL-3.0, GameMaker 8.1 + C++ DLL): [nikitalita/GoldenSunTLA_Editor](https://github.com/nikitalita/GoldenSunTLA_Editor). Its data-table offsets work as a ready-made symbol and struct source.
* **Community hub**: [Golden Sun Hacking Community forum](http://forum.goldensunhacking.net/index.php).

## What this means for us
1. Contribute to goldensun-decomp: set up `camelot-gcc`, pick functions from `asm/`, iterate with asm-differ or decomp.me.
2. Our toolkit needs a `--compiler` profile so it isn't locked to agbcc (README already notes this).
3. Matching compressor for overlays is an open, well-scoped problem that suits a tools person.
4. Later: TLA scaffold using GS1's matched engine code (see 15-developer-profiling.md).

Not verified: exact build flags, the Camelot programmer credits, and whether a JP or EU decomp exists.
