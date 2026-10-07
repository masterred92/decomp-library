# How studios built their games, and what it means for decompiling

Cross-links: 15-developer-profiling.md, 16–19. **[U] = uncertain or unverified, from general knowledge, not a cited source.**

| Studio / game | Language & compiler | Engine reuse / tricks | Decompiler impact |
|---|---|---|---|
| **Camelot**: Golden Sun 1/2 (GBA) | C (Thumb) plus 51 hand-written ARM functions. **Patched gcc-2.96**. Stock m4a audio built with an older agbcc-like compiler ([goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp)) | Map code as compressed overlays loaded into RAM. Hot code copied to IWRAM. Context-Huffman text ([gs-huffman](https://github.com/romhack/GoldenSunCompression/tree/master/gs-huffman)) | You need the right compiler per module, overlays as separate link units, and a matching compressor |
| **Capcom/Flagship with Nintendo**: Minish Cap (GBA) | C via agbcc ([tmc](https://github.com/zeldaret/tmc)) | Engine shared with Four Swords. 3D hitboxes. Leftover flag names and debug strings ([FAQ](https://zelda.deco.mp/games/tmc)) | Matched sibling gives a template for Four Swords |
| **Webfoot**: DBZ LoG/Buu's Fury (GBA) | **ARM Developer Suite 1.2** (Buu's Fury, [buusfury](https://github.com/2genkidev/buusfury)) | Compresses nearly everything, JCALG1-style ([Arefu/Legacy](https://github.com/Arefu/Legacy)) | GCC/agbcc can't match. Needs ADS or a recomp route |
| **Griptonite**: HP GBC | Hand-written SM83 asm (typical of GBC, [U] for this exact title) | Banked code ([HPSS](https://github.com/terinjokes/HPSS-Disassembly)) | Disassembly, not decomp |
| **Game Freak**: Pokémon GBA | C via agbcc (pret's recreation, [agbcc](https://github.com/pret/agbcc)) | m4a/Sappy sound engine, shared across many GBA games | pret methodology is the reference. agbcc is the default first guess |
| **Intelligent Systems**: Fire Emblem 8 (GBA) | C via agbcc. JP build needs an `-mjp-promote` flag variant ([fireemblem8u](https://github.com/laqieer/fireemblem8u), [fireemblem8j](https://github.com/laqieer/fireemblem8j)) | US and JP from the same source, re-linked | Region builds can differ by compiler flags. Match the US version first, then port |
| **Naughty Dog**: Jak & Daxter (PS2) | >98% in **GOAL**, an in-house Lisp with live code reloading ([OpenGOAL](https://github.com/open-goal/jak-project)) | Inlined vector/matrix helpers, `as-type` casts ([progress report](https://github.com/open-goal/open-goal.github.io/blob/master/blog/progress-report-jan-2025/index.mdx)) | Generic C decompilers fail. OpenGOAL wrote a GOAL-specific decompiler plus a new compiler. A custom language needs custom tooling |
| **Rare**: N64 era | C with MIPS IDO [U] | Heavy in-house compression and engine reuse across titles [U] | Check community decomps before assuming anything |
| **Factor 5**: Rogue Squadron (N64/GC) | Known for custom RSP microcode and MusyX audio [U] | Low-level hardware code | Expect hand-written microcode and asm islands. Not cited yet |

## Patterns
1. **Identify the compiler first.** Western GBA studios often used ARM's SDT/ADS. Japanese studios mostly used Nintendo's GCC (agbcc). Exceptions exist (Camelot).
2. **Shared middleware** (m4a, Sappy) is free progress, because it's already matched in other decomps.
3. **RAM-loaded code** (IWRAM, overlays) needs its own link addresses.
4. **Custom compression** usually blocks data and shiftability work before it blocks code.
5. **Debug leftovers** (flag names, strings, debug menus on TCRF) are cheap symbol sources.
6. **Custom languages** (GOAL) mean writing your own decompiler, which is the biggest lift.
