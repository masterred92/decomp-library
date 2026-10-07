# Developer Profiling: Approach a Game by Who Made It
Idea: games from the same studio/team share engines, libraries, compilers, flags, and coding idioms. Matched code and recovered names from a sibling game are often the single biggest accelerator.

## 1. Shared engines & code across a studio's games (documented examples)
- **Nintendo EAD: OoT → MM.** Majora's Mask reused OoT's engine and assets ( https://en.wikipedia.org/wiki/The_Legend_of_Zelda:_Majora%27s_Mask ). zeldaret runs both decomps (https://github.com/zeldaret/oot , https://github.com/zeldaret/mm) with shared conventions and ZAPD tooling, and actor/system names carry over.
- **zeldaret's broader family:** Animal Forest (https://github.com/zeldaret/af) is on the same org/toolchain conventions. Plus OoT GC/VC emulator decomps (oot-gc, oot-vc on decomp.dev).
- **Intelligent Systems: PM64 → TTYD → SPM.** Shared EVT scripting lineage (see 14-paper-mario.md).
- **Snowboard Kids → Snowboard Kids 2** (Racdym): the author reports SK1 shares "many of the quirks" patched in SK2's recomp, and SK1 went from 0 to 100% in 84 days vs 596 for SK2, partly from accumulated tooling and knowledge. https://blog.chrislewis.au/decompiling-a-nintendo-64-game-in-84-days/
- **Platform SDKs are shared by everyone:** N64 libultra/libmus. Chris Lewis notes "more than a hundred source segments from Nintendo's libultra" in SK1, identifiable with N64Sym (same post). GC/Wii Dolphin SDK, MSL, and NW4R are matched once and reused (e.g. TTYD's `libs/dolsdk2004`).
- **Rare** (Banjo, DK64, JFG, Conker): JFG and Conker decomps are on decomp.dev (14.5%, 21.8%). I haven't verified claims of shared Rare engine code, so check each project's docs and Discord before assuming.

## 2. Compiler & flag habits
- Each studio usually locked one toolchain per project. Find it via SDK version strings in the binary, compiler-specific codegen patterns, or existing decomps of the same studio/era. Examples (verified in repos): PM64 = custom GCC + gcc 2.7.2 + IDO 5.3; TTYD = MWCC GC/2.6 (+ GC/1.2.5n for SDK); SPM = GC/3.0a5.2.
- **Per-file flags differ** (SDK vs game code, -O levels, `-inline` settings). dtk `configure.py` lets you set `mw_version`/cflags per library, so copy the sibling project's table as your starting hypothesis.
- Compiler-version detection tools/communities: decomp.me presets list known compilers per platform (https://decomp.me/).

## 3. Recovering names
- **Debug symbols/map files:** some discs/demos shipped with .map files or symbols. ttyd-utils converts .MAP files from other TTYD versions (e.g. the JP demo) into symbol tables (https://github.com/jdaster64/ttyd-utils). Always search for `*.map`, `.elf`, and leftover debug builds and prototypes.
- **Asserts & strings:** assert macros embed file names, line numbers, and expressions (`ASSERT(i < MAX_SCRIPTS)` style in PM64 source). Use Ghidra string search for `.c`/`.cpp` paths to recover the original file layout (split boundaries).
- **C++ RTTI / mangled names** (GC/Wii/PC): MWCC/MSVC leave class names in vtables/RTTI.
- **SDK signatures:** N64Sym (N64), Ghidra FunctionID/FLIRT-style signatures for SDK libs.
- **Community docs:** wikis, TCRF prototype pages (https://tcrf.net/), speedrun/modding docs.

## 4. Reusing matched code from sibling games
1. Diff sibling binaries with matching function hashes. objdiff/dtk and Ghidra Version Tracking (built into Ghidra) map functions between programs.
2. Port the C. It often matches as-is if compiler/flags are identical. Otherwise it's a strong draft for m2c/LLM refinement.
3. Port headers and structs first (actor structs, script VM types). That's where most value carries.
4. Respect licences and attribution between repos, and ask maintainers.

## 5. Checklist: build a developer profile before starting
- [ ] Developer/publisher/year, lead programmers credited (MobyGames/Wikipedia credits), prior and next games by the same team
- [ ] Engine: in-house vs licensed (Unreal/Unity/RenderWare/etc.), and any sibling decomps on decomp.dev
- [ ] Platform SDK version strings, and middleware (audio: libmus/MusyX/CRI; physics; video)
- [ ] Compiler + version + flags hypothesis, checked against sibling projects and decomp.me presets
- [ ] Symbol sources: map files, demos/prototypes, debug builds, other regions, RTTI, asserts
- [ ] Scripting VMs/data formats shared across the series (e.g. EVT)
- [ ] Existing community tools/docs (modding wikis, ttyd-tools-style repos)
- [ ] Which versions/regions exist and which has the most symbols (pick the decomp target accordingly)
- [ ] Legal: own copy, asset-free repo (see 08)
