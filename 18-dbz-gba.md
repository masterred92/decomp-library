# Dragon Ball Z on GBA

Researched 2026-10-07.

## Developers
* **Legacy of Goku I (2002), II (2003), Buu's Fury (2004)**: Webfoot Technologies, published by Infogrames ([Wikipedia](https://en.wikipedia.org/wiki/Dragon_Ball_Z:_The_Legacy_of_Goku_(series))).
* **Supersonic Warriors (2004)**: Arc System Works with Cavia, for Banpresto ([Wikipedia](https://en.wikipedia.org/wiki/Dragon_Ball_Z:_Supersonic_Warriors), [TCRF](https://tcrf.net/Dragon_Ball_Z:_Supersonic_Warriors)).
* Others (Taiketsu, GT: Transformation, etc.) not researched yet.

## Existing work
| Project | Game | Notes |
|---|---|---|
| [2genkidev/buusfury](https://github.com/2genkidev/buusfury) | Buu's Fury | Disassembly that rebuilds `baserom.gba` SHA1 `f1c4b07554d2a3b1ad2f325307051e775ce68087`. **Built with ARM Developer Suite 1.2** (`armasm`/`armcpp`/`armlink`), not GCC. One contributor, last push 2025-07 |
| [mstan/DragonBallZBuusFuryRecomp](https://github.com/mstan/DragonBallZBuusFuryRecomp) | Buu's Fury | Static recompilation to Windows with optional widescreen (v0.0.1 preview), built on a reusable `gbarecomp` framework. BYO ROM and BIOS |
| [spicybung/DragonByteZ](https://github.com/spicybung/DragonByteZ) | LoG I (EU), LoG II (EU/US), Buu's Fury (US) | MIT analyser: header, graphics, soundtrack, decompress. Created 2026-07 |
| [Arefu/Legacy](https://github.com/Arefu/Legacy) | all three Webfoot games | Compression research (wiki). Notes that "they compress… basically everything" |
| [luatsenpai gist](https://gist.github.com/luatsenpai/b3e48b093100e553f61d8824cf64f273) | Buu's Fury | JCALG1-based text decompress, repack and repoint tool |
| TCRF | Supersonic Warriors | Debug menu reachable with code `03000891:0C` (English). No disasm or decomp found |

## Matching implications
* Webfoot used **ARM's ADS compiler**, so agbcc and GCC won't match. A matching C decomp needs ADS 1.2 (proprietary, period tooling) or has to accept a non-matching or recomp approach. Uncertain whether LoG I/II also used ADS. Likely the same studio pipeline, but unverified.
* Heavy JCALG1-style compression means data extraction tools come before code work.

## Contribute vs start fresh
* **Buu's Fury**: contribute to buusfury (C decomp on top of the disasm) or the recomp.
* **LoG I/II**: start fresh with our toolkit, reusing Buu's Fury engine knowledge. Same studio, likely shared engine (unverified).
* **Supersonic Warriors**: open field, but nothing to build on and an unknown compiler.
