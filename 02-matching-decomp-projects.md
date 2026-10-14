# Matching Decompilation Projects
"Matching" means writing C/C++ that compiles **byte-for-byte identical** to the original binary using the original compiler (e.g. IDO for N64, MWCC for GC/Wii, agbcc for GBA). That gives a provable correctness check.

| Project | Platform | Notes | Link |
|---|---|---|---|
| Super Mario 64 (n64decomp/sm64) | N64 / IDO | Fully matching; spawned PC ports and huge mod scene | https://github.com/n64decomp/sm64 |
| Ocarina of Time (zeldaret/oot) | N64 | Assets extracted with ZAPD | https://github.com/zeldaret/oot |
| Majora's Mask (zeldaret/mm) | N64 | Same org/tooling | https://github.com/zeldaret/mm |
| ZAPD | Tool | Zelda Asset Processor for Decomp: XML-driven asset extraction/rebuild | https://github.com/zeldaret/ZAPD |
| pret (pokered, pokeemerald, etc.) | GB/GBC/GBA | Disassemblies (GB) and decomps (GBA, agbcc) | https://github.com/pret |
| Paper Mario (pmret/papermario) | N64 | Matching decomp | https://github.com/pmret/papermario |
| Melee (doldecomp/melee) | GameCube / MWCC | **100% matched (2026-09-07)**; final PR [#3409](https://github.com/doldecomp/melee/pull/3409) labelled ai-assisted | https://github.com/doldecomp/melee |
| Twilight Princess (zeldaret/tp) | GameCube | In progress | https://github.com/zeldaret/tp |
| PS2 decomps | PS2 / EE GCC | e.g. Sly Cooper (TheOnlyZac/sly1); see Decomp.dev progress tracker | https://github.com/TheOnlyZac/sly1 , https://decomp.dev/ |

Progress for many projects: https://decomp.dev/ . Community hub: https://decomp.me/ and https://github.com/zeldaret.

## Expanded notes (2026-10-14)

> **Tutor's note.** A project is "matching" when the build script compiles the C, links it and then
> compares the result against a hash of the original. If one byte differs, the build fails. So
> "64% matched" means 64% of the functions are real C. The rest are still assembly, which gets assembled
> back to the same bytes. The ROM rebuilds perfectly at *every* percentage. Only how much of it is readable C changes.

### Three kinds of "decomp project"
| Kind | What's in the repo | Example | Check |
|---|---|---|---|
| **Disassembly** | Assembly with labels, rebuilt by an assembler | GB/GBC pret repos, [HPSS](19-harry-potter-gbc.md) | Byte-identical ROM |
| **Matching decomp** | C (plus leftover asm) built with the *original* compiler | [pokeemerald](https://github.com/pret/pokeemerald), [tmc](https://github.com/zeldaret/tmc), [goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp) | Byte-identical ROM |
| **Non-matching / functional** | C/C++ that *behaves* the same | OpenGOAL ([20](20-developer-approaches.md)) | Plays the same; tests |

Static recompilation ([03](03-static-recompilation.md)) is a fourth thing. Its output is generated code, not source anyone wrote.

### GBA matching projects worth knowing
| Project | Compiler | Why it's useful to us | Link |
|---|---|---|---|
| pokeemerald / pokefirered / pokeruby | agbcc | The reference GBA methodology (Makefile, `INCLUDE_ASM`, `make compare`) | [pokeemerald](https://github.com/pret/pokeemerald), [pokefirered](https://github.com/pret/pokefirered), [pokeruby](https://github.com/pret/pokeruby) |
| zeldaret/tmc (Minish Cap) | agbcc | A finished non-Pokémon GBA decomp; see [17](17-zelda.md) | [tmc](https://github.com/zeldaret/tmc) |
| fireemblem8u | agbcc | Region builds that differ by compiler flags | [fireemblem8u](https://github.com/FireEmblemUniverse/fireemblem8u) |
| Sonic Advance 3 (SAT-R) | agbcc-family | A non-Nintendo-published GBA target | [sa3](https://github.com/SAT-R/sa3) |
| goldensun-decomp | patched gcc 2.96 | Our project; see [16](16-golden-sun.md) and [25](25-golden-sun-matching-lessons.md) | [goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp) |

### Milestone: Melee at 100%
Melee was one of the biggest GameCube decomps, and it finished on 2026-09-07. The last PR
([#3409](https://github.com/doldecomp/melee/pull/3409)) carries an *ai-assisted* label. **Be careful with this claim:**
the label tells us AI helped with the *final* PR. It doesn't tell us how much of the years of earlier work was AI.
Most of Melee was matched by hand before AI tools were good enough. See [05](05-ai-llm-decompilation.md).

### Common mistakes when reading a project's status
* **Treating % matched as % done.** Naming, documenting and making the code relocatable ("shiftable") come after matching.
* **Treating a fakematch as a real match.** Fakematches are byte-identical but written with tricks; projects track them separately ([25 §3](25-golden-sun-matching-lessons.md)).
* **Assuming the same compiler across a platform.** Golden Sun isn't agbcc, and Webfoot's DBZ games use ARM ADS ([18](18-dbz-gba.md)).
