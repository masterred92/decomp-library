# 21 — Learning Paths: Broad Coding Knowledge for Decomp + AI

Tutor-style plan: learn *with* me, one stage at a time. Each stage says **why it matters for decompilation and AI-assisted coding**. Don't rush; finish a small project at each stage and commit it.

## Stage 0 — Map of the territory (week 1)
- [kamranahmedse/developer-roadmap](https://github.com/kamranahmedse/developer-roadmap) — visual roadmaps; pick the "Computer Science" and "C++" ones.
- [ossu/computer-science](https://github.com/ossu/computer-science) — a full free CS degree curriculum; our long-term spine.
- [EbookFoundation/free-programming-books](https://github.com/EbookFoundation/free-programming-books) — free books for every language here.
- [sindresorhus/awesome](https://github.com/sindresorhus/awesome) — index of every "awesome" list.
- [jlevy/the-art-of-command-line](https://github.com/jlevy/the-art-of-command-line) — the shell skills every tool here assumes.
- [Developer-Y/cs-video-courses](https://github.com/Developer-Y/cs-video-courses) — university lecture videos by topic.
**Why:** decomp touches compilers, CPUs, OSes and languages at once; a map stops you getting lost.

## Stage 1 — C, the language games were written in
- [TheAlgorithms/C](https://github.com/TheAlgorithms/C) — algorithms in plain C; read them, then compile them.
- [rui314/chibicc](https://github.com/rui314/chibicc) — a tiny C compiler built commit by commit. Shows how C turns into assembly, which is decomp in reverse.
- [DoctorWkt/acwj](https://github.com/DoctorWkt/acwj) — "A Compiler Writing Journey", a step-by-step C compiler.
- [TinyCC/tinycc](https://github.com/TinyCC/tinycc) — a small real C compiler you can read.
- [angrave/SystemProgramming](https://github.com/angrave/SystemProgramming) — C systems programming wikibook.
**Why:** almost every GBA/N64/GC game is C. Reading C fluently is the core skill.
**Project:** write 3 small C functions, compile with `-O2`, read the assembly with `objdump -d`.

## Stage 2 — Assembly and how CPUs think
- [cirosantilli/x86-bare-metal-examples](https://github.com/cirosantilli/x86-bare-metal-examples) — tiny programs running with no OS.
- [mytechnotalent/Reverse-Engineering](https://github.com/mytechnotalent/Reverse-Engineering) — free RE tutorial covering x86, ARM and RISC-V.
- [gbadev-org/awesome-gbadev](https://github.com/gbadev-org/awesome-gbadev) and [gbdev/awesome-gbdev](https://github.com/gbdev/awesome-gbdev) — ARM/GBA and Game Boy resources.
**Why:** decomp means reading assembly and guessing the C behind it.

## Stage 3 — Compilers and interpreters
- [jamiebuilds/the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler) — a whole compiler in about 200 commented lines. Start here.
- [munificent/craftinginterpreters](https://github.com/munificent/craftinginterpreters) — the book *Crafting Interpreters* (free at craftinginterpreters.com).
- [codecrafters-io/build-your-own-x](https://github.com/codecrafters-io/build-your-own-x) — build your own compiler, OS, emulator and more.
**Why:** matching decomp is about predicting compiler output. Knowing how compilers work makes this far easier.

## Stage 4 — Operating systems and low level
- [cfenollosa/os-tutorial](https://github.com/cfenollosa/os-tutorial) — build a tiny OS from scratch.
- [mit-pdos/xv6-riscv](https://github.com/mit-pdos/xv6-riscv) — MIT's small teaching Unix (course 6.1810).
- [remzi-arpacidusseau/ostep-projects](https://github.com/remzi-arpacidusseau/ostep-projects) — projects for the free book *OSTEP*.
- [0xAX/linux-insides](https://github.com/0xAX/linux-insides) — how the Linux kernel works.
- [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat), [SerenityOS/serenity](https://github.com/SerenityOS/serenity) — big real-world references.

## Stage 5 — Python (our scripting glue)
- [Asabeneh/30-Days-Of-Python](https://github.com/Asabeneh/30-Days-Of-Python) — daily beginner course.
- [gto76/python-cheatsheet](https://github.com/gto76/python-cheatsheet), [satwikkansal/wtfpython](https://github.com/satwikkansal/wtfpython) — quick reference and surprising corners.
- [TheAlgorithms/Python](https://github.com/TheAlgorithms/Python).
**Why:** splat, our toolkit and Ghidra scripts are all Python.

## Stage 6 — C++, Rust, Go, JS/TS (breadth)
- C++: [isocpp/CppCoreGuidelines](https://github.com/isocpp/CppCoreGuidelines), [AnthonyCalandra/modern-cpp-features](https://github.com/AnthonyCalandra/modern-cpp-features), [fffaraz/awesome-cpp](https://github.com/fffaraz/awesome-cpp), [TheAlgorithms/C-Plus-Plus](https://github.com/TheAlgorithms/C-Plus-Plus). GameCube/Wii games often use C++.
- Rust: [rust-lang/book](https://github.com/rust-lang/book), [rust-lang/rustlings](https://github.com/rust-lang/rustlings), [google/comprehensive-rust](https://github.com/google/comprehensive-rust), [TheAlgorithms/Rust](https://github.com/TheAlgorithms/Rust). objdiff and decomp-toolkit are written in Rust.
- Go: [quii/learn-go-with-tests](https://github.com/quii/learn-go-with-tests), [inancgumus/learngo](https://github.com/inancgumus/learngo).
- JS/TS: [getify/You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS), [trekhleb/javascript-algorithms](https://github.com/trekhleb/javascript-algorithms), [microsoft/TypeScript](https://github.com/microsoft/TypeScript). Browser ports (like the Halo one) use JS and WebAssembly.
- C#/.NET (Unity): [dotnet/csharplang](https://github.com/dotnet/csharplang).

## Stage 7 — CS fundamentals and practice
- [jwasham/coding-interview-university](https://github.com/jwasham/coding-interview-university), [donnemartin/system-design-primer](https://github.com/donnemartin/system-design-primer), [practical-tutorials/project-based-learning](https://github.com/practical-tutorials/project-based-learning), [freeCodeCamp/freeCodeCamp](https://github.com/freeCodeCamp/freeCodeCamp).

## Stage 8 — LLMs: how they work and how to build with them
1. [karpathy/micrograd](https://github.com/karpathy/micrograd), then [karpathy/nn-zero-to-hero](https://github.com/karpathy/nn-zero-to-hero) (with the YouTube series). Backprop explained from scratch.
2. [karpathy/nanoGPT](https://github.com/karpathy/nanoGPT), [karpathy/minbpe](https://github.com/karpathy/minbpe) (tokenizers), [karpathy/llm.c](https://github.com/karpathy/llm.c) (GPT training in C), [karpathy/LLM101n](https://github.com/karpathy/LLM101n) (course outline).
3. [rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch) — book code: build a GPT step by step in PyTorch.
4. [mlabonne/llm-course](https://github.com/mlabonne/llm-course) — roadmap covering fundamentals, the LLM scientist track and the LLM engineer track.
5. [huggingface/course](https://github.com/huggingface/course), [huggingface/smol-course](https://github.com/huggingface/smol-course), [huggingface/transformers](https://github.com/huggingface/transformers).
6. Beginner: [microsoft/generative-ai-for-beginners](https://github.com/microsoft/generative-ai-for-beginners), [microsoft/AI-For-Beginners](https://github.com/microsoft/AI-For-Beginners), [microsoft/ML-For-Beginners](https://github.com/microsoft/ML-For-Beginners).
7. Using LLMs: [dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide), [anthropics/courses](https://github.com/anthropics/courses), [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) (run models locally).
8. Papers in code: [labmlai/annotated_deep_learning_paper_implementations](https://github.com/labmlai/annotated_deep_learning_paper_implementations).
**Why:** AI-assisted decomp (LLM4Decompile, GhidraMCP) only makes sense once you know what a model actually does.

## Stage 9 — Security practice (legal)
- [pwncollege/dojo](https://github.com/pwncollege/dojo) (pwn.college), [ctf-wiki/ctf-wiki](https://github.com/ctf-wiki/ctf-wiki), [apsdehal/awesome-ctf](https://github.com/apsdehal/awesome-ctf), [OWASP/CheatSheetSeries](https://github.com/OWASP/CheatSheetSeries). See note 23.

## Stage 10 — Game design & writing
Not decomp directly, but decomp is reading other people's *design decisions* in code. These teach what those decisions were for.

**Maps and reading lists**
- [ellisonleao/magictools](https://github.com/ellisonleao/magictools) — the big index of game-dev tools and resources.
- [Kavex/GameDev-Resources](https://github.com/Kavex/GameDev-Resources) and [Calinou/awesome-gamedev](https://github.com/Calinou/awesome-gamedev) — broad resource lists (the second sticks to free software).
- [dawdle-deer/awesome-learn-gamedev](https://github.com/dawdle-deer/awesome-learn-gamedev) — learning-focused: courses, books, talks.
- [Roobyx/awesome-game-design](https://github.com/Roobyx/awesome-game-design) — design theory, books, GDC talks and examples.
- [utilForever/game-developer-roadmap](https://github.com/utilForever/game-developer-roadmap) — a visual roadmap of game-programming skills.
- [gheja/game-design-documents](https://github.com/gheja/game-design-documents) — real design documents from well-known games.

**Programming patterns and game AI**
- [munificent/game-programming-patterns](https://github.com/munificent/game-programming-patterns) — the book *Game Programming Patterns* (free at gameprogrammingpatterns.com). Game loop, update method, component, state: the shapes you will see again in decompiled code.
- [BehaviorTree/BehaviorTree.CPP](https://github.com/BehaviorTree/BehaviorTree.CPP) — behaviour trees in C++.
- [crashkonijn/GOAP](https://github.com/crashkonijn/GOAP) — goal-oriented action planning (the F.E.A.R. approach) for Unity.
- [wangchen/Programming-Game-AI-by-Example-src](https://github.com/wangchen/Programming-Game-AI-by-Example-src) — source for Buckland's classic book: state machines, steering, pathfinding.

**Narrative and interactive fiction tools**
- [inkle/ink](https://github.com/inkle/ink), [inkle/inky](https://github.com/inkle/inky) (editor), [y-lohse/inkjs](https://github.com/y-lohse/inkjs) (web runtime), [inkle/ink-library](https://github.com/inkle/ink-library) (samples) and [inkle/the-intercept](https://github.com/inkle/the-intercept) (a small complete game). ink is the scripting language behind *80 Days* and *Heaven's Vault*. Start here for branching writing.
- [YarnSpinnerTool/YarnSpinner](https://github.com/YarnSpinnerTool/YarnSpinner) — dialogue language used in *Night in the Woods* and *A Short Hike*.
- [klembot/twinejs](https://github.com/klembot/twinejs) and [tmedwards/sugarcube-2](https://github.com/tmedwards/sugarcube-2) — Twine, the fastest way to prototype a branching story in a browser.
- [dfabulich/choicescript](https://github.com/dfabulich/choicescript) — Choice of Games' stat-driven multiple-choice format.
- [ganelson/inform](https://github.com/ganelson/inform) — Inform 7, parser IF written in near-English.
- [nathanhoad/godot_dialogue_manager](https://github.com/nathanhoad/godot_dialogue_manager) — nonlinear dialogue for Godot.
- [galaxykate/tracery](https://github.com/galaxykate/tracery) — tiny story-grammar generator; great first procedural-text toy.
- [JoshuaGrams/tiny-qbn](https://github.com/JoshuaGrams/tiny-qbn) — storylets (quality-based narrative, the *Fallen London* model) for Twine.
- [tajmone/awesome-interactive-fiction](https://github.com/tajmone/awesome-interactive-fiction) — index of IF systems and tools.

**Social simulation and emergent story (research)**
- [ensemble-engine/ensemble](https://github.com/ensemble-engine/ensemble) — rules-based social AI (successor to the *Prom Week* engine).
- [james-owen-ryan/talktown](https://github.com/james-owen-ryan/talktown) — simulated town with generations of characters, memory and gossip.
- [mkremins/winnow](https://github.com/mkremins/winnow) — "story sifting": finding interesting stories inside a simulation log.

**Procedural generation**
- [mxgmn/WaveFunctionCollapse](https://github.com/mxgmn/WaveFunctionCollapse) — generates tilemaps from one example; used in *Caves of Qud* and *Townscaper*.
- [mxgmn/MarkovJunior](https://github.com/mxgmn/MarkovJunior) — the same author's rewrite-rule language, 150+ examples.
- [amitp/mapgen2](https://github.com/amitp/mapgen2) — Red Blob Games' polygon island map generator (read the article on redblobgames.com alongside it).

**Open-source games to study**
- [OpenTTD/OpenTTD](https://github.com/OpenTTD/OpenTTD) — simulation economy, decades of design iteration. It began as a reverse-engineered Transport Tycoon, so it's relevant to decomp too.
- [CleverRaven/Cataclysm-DDA](https://github.com/CleverRaven/Cataclysm-DDA) — huge data-driven (JSON) survival sim; emergent stories come from systems.
- [crawl/crawl](https://github.com/crawl/crawl), [NetHack/NetHack](https://github.com/NetHack/NetHack), [tmewett/BrogueCE](https://github.com/tmewett/BrogueCE) — three roguelike philosophies: depth, legacy and minimalism.
- [wesnoth/wesnoth](https://github.com/wesnoth/wesnoth) — turn-based strategy with a large written campaign library and a scripting language (WML).
- [DFHack/dfhack](https://github.com/DFHack/dfhack) — Dwarf Fortress itself is closed source, but DFHack's reverse-engineered structures show how its simulation is laid out. That's decomp meeting design.

**Why:** knowing *why* a system exists (a state machine for enemy AI, a script VM for dialogue) makes it much faster to recognise in assembly. It's also the base for designing our own games.
**Project:** write a 10-minute branching story in ink (Inky), commit the `.ink` file, then add one storylet-style "quality" check.

## Top 10 starter repos (do these first, in order)
1. ossu/computer-science 2. jlevy/the-art-of-command-line 3. TheAlgorithms/C 4. jamiebuilds/the-super-tiny-compiler 5. rui314/chibicc 6. mytechnotalent/Reverse-Engineering 7. Asabeneh/30-Days-Of-Python 8. karpathy/nn-zero-to-hero 9. rasbt/LLMs-from-scratch 10. mlabonne/llm-course

All links checked on 2026-10-07 (each repo was starred through the GitHub API, which fails for repos that don't exist).
Stage 10 links were checked the same day with `gh api repos/OWNER/REPO`.
