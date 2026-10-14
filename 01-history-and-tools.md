# History & Key Tools
## History (short)
- Cristina Cifuentes' 1994 PhD thesis "Reverse Compilation Techniques" (dcc) set the structured-decompilation foundation. https://yurichev.com/mirrors/DCC_decompilation_thesis.pdf
- Hex-Rays decompiler (Ilfak Guilfanov) shipped as an IDA plugin in 2007, making decompilation mainstream in commercial RE. https://hex-rays.com/
- NSA released Ghidra free/open source in 2019 (RSA Conference), which is the biggest democratising event for hobbyist/game decomp. https://github.com/NationalSecurityAgency/ghidra
- 2019: SM64 decomp went public, and matching decomp became a community movement (see 02).
- 2020s: shared tooling makes matching faster: decomp.me scratches, asm-differ, the permuter, splat, and objdiff with decomp.dev progress tracking (see 04).
- 2024+: LLM-based decompilers (see 05).
- 2026: Melee hits 100% (final PR labelled ai-assisted), agent-driven decomps that use byte-matching oracles get attention, along with fast publisher takedowns, and GBA static recompilers (gbarecomp) appear (see 02, 03, 05, 08).

## Tools
| Tool | What | Notes | Link |
|---|---|---|---|
| IDA Pro + Hex-Rays | Disassembler/decompiler | Commercial industry standard; free tier exists | https://hex-rays.com/ida-pro |
| Ghidra | Free SRE suite + decompiler | Java; many CPUs incl. MIPS/PPC/ARM; scripting in Java/Python | https://github.com/NationalSecurityAgency/ghidra |
| Binary Ninja | Commercial RE platform | BNIL intermediate languages, strong Python API | https://binary.ninja/ |
| RetDec | Open-source LLVM-based decompiler (Avast) | Repo marked limited maintenance | https://github.com/avast/retdec |
| angr | Python binary analysis framework | Symbolic execution, CFG recovery, has a decompiler | https://angr.io/ |
| radare2 / Cutter | CLI RE framework / Qt GUI | Cutter bundles the Ghidra decompiler (rz-ghidra); now on Rizin | https://github.com/radareorg/radare2 , https://cutter.re/ |
| ILSpy | .NET decompiler | Unity Mono games (Assembly-CSharp.dll) | https://github.com/icsharpcode/ILSpy |
| dnSpy / dnSpyEx | .NET debugger + editor | Original archived; dnSpyEx is the maintained fork | https://github.com/dnSpyEx/dnSpy |
| Il2CppDumper | Recovers metadata from Unity IL2CPP builds | Uses global-metadata.dat + GameAssembly; outputs dummy DLLs and scripts for IDA/Ghidra | https://github.com/Perfare/Il2CppDumper |
| jadx | Dex → Java decompiler | Android games/APKs | https://github.com/skylot/jadx |

> **Tutor's note.** A *decompiler* (Ghidra, IDA) gives you C that means the same thing. A *matching decomp* needs C that compiles to the
> same **bytes**. The tools above are the first kind. The matching toolkit is in [04](04-decomp-tooling.md), and why
> matching is harder is explained in [25](25-golden-sun-matching-lessons.md).
