# Platform Notes
- **MIPS / N64 (and PS1/PS2/PSP)**: N64 games often used SGI IDO 5.3/7.1 (run via the IDO static recomp https://github.com/decompals/ido-static-recomp) or GCC (KMC). Watch for delay slots. Tooling: splat, m2c, asm-differ, permuter. PS2 Emotion Engine is MIPS R5900 (128-bit regs, EE-GCC/MWCC/SN).
- **PowerPC / GameCube / Wii (and Xbox 360/PS3)**: Metrowerks CodeWarrior (MWCC) on GC/Wii, with exact version matching critical. Tooling: decomp-toolkit, objdiff, decomp.me. Formats: DOL/REL. 360 is PPC (Xenon), so XenonRecomp.
- **ARM / GBA / DS / 3DS / Switch**: GBA ARM7TDMI with Thumb, agbcc (pret). DS ARM9/ARM7 with MWCC for ARM. Switch is AArch64 with Clang/LLVM, NSO executables; Ghidra/IDA plus loaders. Thumb/ARM mode switching matters.
- **x86 / PC**: MSVC builds; PDBs (if leaked or shipped) are gold. Tools: IDA, Ghidra, Binary Ninja. Matching decomp of MSVC games exists (e.g. objdiff supports PE/COFF). Expect anti-tamper/DRM, which you shouldn't circumvent (see 08).
- **Unity**: Mono builds go to ILSpy/dnSpyEx (near-source C#). IL2CPP builds go to Il2CppDumper (or Cpp2IL https://github.com/SamboyCoding/Cpp2IL) then Ghidra/IDA. Assets: AssetRipper https://github.com/AssetRipper/AssetRipper
- **Unreal**: native C++. The reflection system (UObject/UClass names) leaks lots of symbol info; use SDK dumpers and UE source (via Epic's GitHub access) for struct layouts. Assets in .pak/IoStore: FModel https://fmodel.app/

## GBA and Thumb, in depth (added 2026-10-14)

> **Tutor's note.** You don't need to *write* assembly to match functions. You need to *read* it well enough to
> see where your output differs from the target. These are the GBA facts that keep coming up.

### The machine
* **CPU:** ARM7TDMI (ARMv4T) at ~16.78 MHz. It has two instruction sets: **ARM** (32-bit instructions) and **Thumb** (16-bit).
  Most GBA game code is Thumb, because the cartridge bus is 16 bits wide and Thumb runs faster from ROM.
* **Memory map** (from [GBATEK](https://problemkaputt.de/gbatek.htm) and [Tonc](https://www.coranac.com/tonc/text/toc.htm)):
  ROM at `0x08000000`, 256 KB **EWRAM** at `0x02000000` (slower 16-bit bus), 32 KB **IWRAM** at `0x03000000`
  (fast 32-bit bus). Hot loops are often copied into IWRAM and run as ARM code there.
* Code that's *loaded* into RAM (Golden Sun's map overlays, whose function names contain `2008…`) has to be linked at its RAM address,
  not its ROM address ([16](16-golden-sun.md), [20](20-developer-approaches.md)).

### Registers and the calling convention
* **Thumb limits:** most instructions can only use the **low registers r0–r7**, so register pressure is high and small
  wording changes shuffle the allocation (the most common miss in [25](25-golden-sun-matching-lessons.md)).
* **The ARM convention** ([AAPCS, ARM-software/abi-aa](https://github.com/ARM-software/abi-aa)): **r0–r3** carry
  arguments, **r0** carries the return value, r4–r11 are callee-saved, and `lr` holds the return address.
* **Compiler flags can change that.** Golden Sun's `-fcall-used-r4` makes r4 scratch, so long-lived values move to r5 and up
  ([25 §1](25-golden-sun-matching-lessons.md); [GCC code-gen options](https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html)).

### Thumb "tells" you can read off the code
| You see | It usually means |
|---|---|
| `ldrsh` / `ldrsb` | The field is **signed** (`s16`/`s8`) |
| `ldrh` / `ldrb` | The field is **unsigned** |
| `lsl` then `asr`/`lsr` by the same amount | The compiler is narrowing a value to a smaller type (choose your temp's type carefully) |
| Function ends `pop {r0}; bx r0` | gcc 2.96 thinks the function is **void** |
| Function ends `pop {r1}; bx r1` | It thinks the function **returns a value** (even if unused) ([25 §2.9](25-golden-sun-matching-lessons.md)) |
| `bl _call_via_r3` (or another register) | An indirect call through a function pointer; ARMv4T Thumb needs this helper |

(On ARMv4T, `pop {pc}` can't switch back to ARM mode, so compilers that support interworking pop into a low register and `bx`.)

### Which compiler? (GBA)
| Compiler | Used by | Notes |
|---|---|---|
| **agbcc** (Nintendo's GCC 2.95-era build, recreated by pret) | Most Nintendo-published and many Japanese titles | [pret/agbcc](https://github.com/pret/agbcc) |
| **Patched gcc 2.96** | Camelot (Golden Sun 1/2) | [16](16-golden-sun.md) |
| **ARM SDT / ADS 1.2 (armcc)** | Many Western studios (e.g. Webfoot's DBZ) | GCC can't match it; see [18](18-dbz-gba.md) |

### Common mistakes (GBA)
* **Assuming agbcc.** Check first; [15](15-developer-profiling.md) explains how.
* **Matching ARM code with Thumb flags** (or the other way round). Check which mode the function is in before drafting.
* **Ignoring alignment at section ends**, where gas may pad with a NOP instead of zeros ([25 §1](25-golden-sun-matching-lessons.md)).
* **Using `u16` everywhere.** The load instruction tells you the signedness.
