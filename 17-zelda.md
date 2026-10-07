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
