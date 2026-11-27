# Decompilation Reference Library
Built 2026-10-07. Goal: grow toward decompiling games. Sources linked in each file; verify before relying on version-specific details. How the later lessons changed over time is in [34-learning-timeline.md](34-learning-timeline.md); individual rules, including superseded ones, are in [INDEX.md](INDEX.md).

| File | Topic |
|---|---|
| [01-history-and-tools.md](01-history-and-tools.md) | History; IDA/Hex-Rays, Ghidra, Binary Ninja, RetDec, angr, radare2/Cutter, ILSpy/dnSpy, Il2CppDumper, jadx; timeline to 2026 |
| [02-matching-decomp-projects.md](02-matching-decomp-projects.md) | SM64, OoT/MM (ZAPD), pret, Paper Mario, Melee, Twilight Princess, PS2; Melee 100%, project kinds, GBA project table |
| [03-static-recompilation.md](03-static-recompilation.md) | N64Recomp, XenonRecomp, Ship of Harkinian; how gbarecomp works, decomp vs recomp |
| [04-decomp-tooling.md](04-decomp-tooling.md) | decomp.me, m2c, asm-differ, splat, objdiff, permuter; GBA splitters, permuter/differ in depth, common mistakes |
| [05-ai-llm-decompilation.md](05-ai-llm-decompilation.md) | LLM4Decompile and AI-assisted RE; byte-oracle lesson, contested claims, 2026 takedowns |
| [06-platform-notes.md](06-platform-notes.md) | MIPS/N64, PPC/GC/Wii, ARM/GBA/DS/Switch, x86/PC, Unity/Unreal; GBA/Thumb deep dive (memory map, calling convention, codegen tells) |
| [07-workflow.md](07-workflow.md) | Step-by-step game decomp workflow; worked example from PR #34, checklist, common mistakes |
| [08-legal-ethics.md](08-legal-ethics.md) | Legal and ethical notes (not legal advice); safe repo shape, 2026 AI takedowns |
| [09-box-tool-availability.md](09-box-tool-availability.md) | What's installed on this box |
| [10-media-notes.md](10-media-notes.md) | User-shared videos |
| [11-desktop-apps-photoshop.md](11-desktop-apps-photoshop.md) | Non-game apps: Photoshop news, C++ RE, PSD, legal |
| [12-tracking-new-projects.md](12-tracking-new-projects.md) | Where to find new projects |
| [13-candidate-projects.md](13-candidate-projects.md) | Ranked shortlist of projects to work on |
| [14-paper-mario.md](14-paper-mario.md) | Paper Mario decomp/port status |
| [15-developer-profiling.md](15-developer-profiling.md) | Studio/developer profiling and code reuse |
- [16-golden-sun.md](16-golden-sun.md)
- [17-zelda.md](17-zelda.md)
- [18-dbz-gba.md](18-dbz-gba.md)
- [19-harry-potter-gbc.md](19-harry-potter-gbc.md)
- [20-developer-approaches.md](20-developer-approaches.md)
- [21-learning-paths.md](21-learning-paths.md) — study plan: CS, C, assembly, compilers, OS, Python/C++/Rust/Go/JS, LLMs
- [22-language-history.md](22-language-history.md) — history of programming languages, why C won, human lingua francas, LLMs as translators
- [23-weird-web.md](23-weird-web.md) — weird open-source websites: URL-bar/favicon games, Doom in a PDF and in CSS, browser emulators, WASM ports of decompiled games, CSS art, tiny demos
- [24-linux-and-torvalds.md](24-linux-and-torvalds.md) — Linux history (1991 post, GPL, Just for Fun), how Git began, kernel development process, torvalds repos, GNU toolchain/ELF/syscalls for decomp, Linux and OS learning repos
- [25-golden-sun-matching-lessons.md](25-golden-sun-matching-lessons.md) — lessons from matching 103 Golden Sun functions (PRs #32–#54): gcc 2.96 quirks, write-order and struct patterns, the Thumb return-type tell, misses, struct layouts, PR workflow, Camelot's coding style
- [26-what-decomp-teaches-about-programming.md](26-what-decomp-teaches-about-programming.md) — general programming lessons from matching 103 Golden Sun functions: compilers, data layout, types, debugging, verification, automation, collaboration
- [27-lessons-from-addon-releases.md](27-lessons-from-addon-releases.md) — November 2026 roundup: shared-lib versioning, client restrictions, tests you can prove, CI on archived repos, release and store hygiene, public-repo hygiene, team review
- [28-tests-mocks-and-lua-pitfalls.md](28-tests-mocks-and-lua-pitfalls.md) — Lua and/or traps, truthful mocks, live-state renders, quantity-scaling checks, proving a test by reverting it
- [29-honest-data.md](29-honest-data.md) — controls must change outputs, cite the real publisher, totals match their parts, byte-compare data rebuilds, do not invent missing numbers
- [30-hidden-data-and-addon-apis.md](30-hidden-data-and-addon-apis.md) — combat hiding is the default, runtime detection, feature-test missing APIs, combat-safe keyboard UI
- [31-ui-contrast-and-layout.md](31-ui-contrast-and-layout.md) — contrast over bright worlds and glows, icons at 64px, move a colliding status, freeze height only after sizing for combat
- [32-auction-house-pitfalls.md](32-auction-house-pitfalls.md) — live deposits, 2/8/24 hour listings, separate neutral-house prices, incomplete scans
- [33-release-ci-and-collaboration.md](33-release-ci-and-collaboration.md) — re-read live store pages, pin credential-handling actions, one writer per checkout
- [34-learning-timeline.md](34-learning-timeline.md) — October–November 2026 chronology, from the decomp library through addon and data lessons
- [35-cross-project-decompiler-lessons.md](35-cross-project-decompiler-lessons.md) — cross-project matching architecture, FAKEMATCH ladder, categorized ASM-to-C patterns, a playbook for future decomp repos, and a checklist for our own decompiler
- [INDEX.md](INDEX.md) — lesson index, including superseded rules and the dates they changed
