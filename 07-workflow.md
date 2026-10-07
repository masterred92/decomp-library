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

Learning: decomp.me tutorials, https://github.com/zeldaret/oot/blob/main/docs/tutorial/contents.md , pret wiki.
