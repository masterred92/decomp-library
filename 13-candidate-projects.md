# Candidate Projects (ranked shortlist, 2026-10-07)
Progress % is **matched code %** from decomp.dev's public feed (https://decomp.dev/projects.json, fetched 2026-10-07). It changes daily. Compiler info comes from platform convention (N64 = IDO/KMC GCC, GC/Wii = MWCC, PS2 = EE-GCC/MWCC), so **confirm in each repo's README**. Legal baseline for all of them: dump your own copy, commit no ROMs/assets, and follow the project's contribution rules (see 08).

## Context: AI-assisted matching decomp is now proven
- Chris Lewis took Snowboard Kids 2 (596 days) and Snowboard Kids (**84 days**) to 100% with coding agents + m2c + permuter + decomp.me. Blanket-running m2c alone matched only 17/1,830 functions. https://blog.chrislewis.au/decompiling-a-nintendo-64-game-in-84-days/ , https://blog.chrislewis.au/using-coding-agents-to-decompile-nintendo-64-games/
- Benchmark: a Claude → compiler → objdiff loop matched **74% of 60 functions** from Sonic Advance 3 (GBA) and Animal Forest (N64). m2c and the permuter helped on IDO code. https://macabeus.medium.com/can-llms-really-do-matching-decompilation-i-tested-60-functions-to-find-out-4e39b0ae4288
- decomp.dev lists zcanann/bfbb, labelled "(Fork: AI)", at 93.9% vs upstream bfbbdecomp/bfbb at 38.1%. Check how that fork's matches were produced before relying on it.

## Ranked shortlist
| # | Cat | Project | Platform/CPU | Compiler | Progress | Why it fits our toolkit | Difficulty |
|---|---|---|---|---|---|---|---|
| 1 | a/b | **Animal Forest** https://github.com/zeldaret/af | N64 / MIPS R4300 | IDO (zeldaret convention) | 18.5% | It's the AI benchmark target, and m2c + permuter work well on IDO. zeldaret has docs and a Discord. splat-style tooling + mips cross binutils | Medium |
| 2 | a | **Super Mario Sunshine** https://github.com/doldecomp/sms | GC / PPC Gekko | MWCC | 47.8% | Lots of unmatched functions. objdiff/decomp.me workflow; Ghidra + powerpc binutils | Medium |
| 3 | a | **Paper Mario: TTYD** https://github.com/doldecomp/ttyd | GC / PPC | MWCC | 12.1% | Early, big codebase, active doldecomp org | Medium |
| 4 | a | **Wind Waker** https://github.com/zeldaret/tww | GC / PPC | MWCC | 79.0% | Mature docs. Remaining functions are harder (C++), so good for learning objdiff | Med-Hard |
| 5 | a | **pret (pokeemerald etc.)** https://github.com/pret/pokeemerald | GBA / ARM7TDMI | agbcc | (repo states completion; pret focus is now documentation) | The easiest on-ramp is naming/docs PRs. Needs `gcc-arm-none-eabi` (not yet installed) | Easy |
| 6 | b | **Wave Race 64** https://github.com/LLONSIT/Wave-Race-64 | N64 / MIPS | check README | 33.7% | Mid-stage N64 project, fits the agent loop + m2c/permuter | Medium |
| 7 | b | **Jet Force Gemini** https://github.com/Ryan-Myers/Jet-Force-Gemini | N64 / MIPS | check README | 14.5% | Rare-era N64 with lots left to do | Med-Hard |
| 8 | b | **Glover** https://github.com/bigyoshi51/glover-decomp | N64 / MIPS | check README | 1.0% | Almost greenfield. Do library (libultra) matching first, as in the Snowboard Kids write-up | Medium |
| 9 | b | **Sly Cooper** https://github.com/TheOnlyZac/sly1 | PS2 / MIPS R5900 | check README | 7.9% | PS2 support in splat/m2c. Well-documented project | Hard |
| 10 | c | **Snowboard Kids recomp** (in progress per author) / existing **Snowboard Kids 2: Recompiled** https://github.com/cdlewis/snowboardkids2-recomp | N64 → native | N64Recomp | SK2 recomp released; SK1 "underway" | Reference template for decomp → N64Recomp port. Contribute or study | Medium |
| 11 | c | **Dr. Mario 64** https://github.com/AngheloAlf/drmario64 / **Pilotwings 64** https://github.com/gcsmith/Pilotwings64Decomp | N64 | IDO/GCC per README | 100% | Fully matched decomps are natural N64Recomp candidates. **First check whether a recomp already exists** (I haven't verified) | Med-Hard |
| 12 | c | **XenonRecomp target study** https://github.com/hedge-dev/XenonRecomp | X360 / PPC | n/a | tool | Learn from Unleashed Recompiled. Needs Clang 18+ (box has 19) | Hard |
| 13 | d | **Unity Mono game you own + BepInEx/Harmony** https://github.com/BepInEx/BepInEx , https://github.com/pardeike/Harmony | x86-64 / .NET | C# (Mono) | n/a | ilspycmd gives near-source C#. Mods patch at runtime with no redistribution of game code | Easy |
| 14 | e | **psd-tools** https://github.com/psd-tools/psd-tools (contribute) / **PhotoCraft** https://github.com/storytold/photocraft (issues/tests) | PSD format | n/a | active | Clean-room interop from Adobe's public PSD spec. Uses binary diffing, not decompilation | Easy-Med |

## Legal notes per category
- **a/b/c:** asset-free matching decomp repos are the community norm. Nintendo has historically tolerated them but that isn't guaranteed. Recomps require the user's own ROM.
- **d:** don't redistribute decompiled game code, and avoid online/anti-cheat games. Check the game's EULA and modding policy.
- **e:** work from public specs and black-box behaviour. Don't use decompiled Adobe code (EULA). Keep a clean-room record.

## Where this came from / how to refresh
`curl -s https://decomp.dev/projects.json | python3 -c '...'` (see 12-tracking-new-projects.md). Re-rank monthly.
