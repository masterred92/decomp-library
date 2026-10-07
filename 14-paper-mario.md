# Paper Mario Decomps: Status (checked 2026-10-07)

## Paper Mario (N64): pmret/papermario. **Done (100%)**
- Repo: https://github.com/pmret/papermario · site https://papermar.io/ · Discord https://discord.gg/PgcMpQTzh5 (from README)
- Progress (papermar.io shield JSON, today): **US 100.00%, PAL 100.00%, iQue 100.00%, JP 100.17%** (the badge reads over 100%). The site says: "We have reached 100% on the US, PAL, and iQue releases!"
- Regions/ROM SHA1s (README): US `3837f44c…97a0`, JP `b9cca3ff…86d3`, PAL `2111d392…f9e1`, iQue `5c724685…8582`. At least one baserom goes in `ver/<region>/baserom.z64`.
- Compilers (from `install_compilers.sh`): a custom **pmret/gcc-papermario + binutils-papermario** (GCC-based MIPS toolchain), plus **decompals mips-gcc-2.7.2 / binutils-2.6** and **IDO 5.3** (via ido-static-recomp). The script downloads them all.
- Build reqs (SETUP.md): Linux/macOS/WSL2/Nix; `./install_deps.sh`, `./install_compilers.sh`; Rust + `cargo install pigment64` (image tool); then build per SETUP.md. Contribution guide: CONTRIBUTING.md.
- **PC port:** **PaperBoat** by Harbour Masters, released Sept 2026. It's based on the **PaperMario-DX** decomp, needs a legally dumped **US** ROM (checker at https://paperboat.equipment/), and has DX11/OpenGL/Metal backends. https://github.com/HarbourMasters/PaperBoat , https://www.gamingonlinux.com/2026/09/harbour-masters-release-paperboat-their-paper-mario-64-pc-port/ . GamingOnLinux says an earlier port, "Paper Mario ReCut", seems inactive. I didn't find an N64Recomp-based recomp.
- What's left for contributors: documentation and naming, plus mods and ports, not matching.

## Paper Mario: The Thousand-Year Door (GameCube): doldecomp/ttyd. **Early (12.1%)**
- Repo: https://github.com/doldecomp/ttyd · progress https://decomp.dev/doldecomp/ttyd · Discord https://discord.gg/hKx3FJJgrV (README badge; the server ID matches the GC/Wii decomp server)
- decomp.dev today: **12.1% code matched, 20.8% functions matched**. Only version is **G8MJ01 = Rev 0 (JP)**.
- Toolchain (configure.py): decomp-toolkit **v1.8.3**, objdiff **v3.7.1**, linker **MWCC GC/2.6**. Game code and RELs use **GC/2.6**; the Dolphin SDK libs use **GC/1.2.5n**. On Linux, wibo runs the Windows MWCC automatically. You need Python + ninja.
- Setup: put your disc image in `orig/G8MJ01/` (ISO/RVZ/WIA/WBFS/CISO/NFS/GCZ/TGC), then `python configure.py && ninja`, then open objdiff on the generated `objdiff.json`.
- README quirks: it still has dtk-template leftovers (the build badge points to zeldaret/tww, and the clone URL is NWPlayer123/G8MJ01-dtk), and one line wrongly calls G8MJ01 "USA". There's no CONTRIBUTING.md (404), so ask on the Discord.
- Repo has **no assets or assembly**, so you must supply your own JP disc. A US/PAL copy won't work yet.

## Super Paper Mario (Wii): SeekyCt/spm-decomp. **Partial by design (2.4%)**
- https://github.com/SeekyCt/spm-decomp · docs https://github.com/SeekyCt/spm-docs · Discord https://discord.gg/ndrxwcyCum
- decomp.dev: 2.4% code / 3.8% functions. Versions EU0, EU1, JP0, KR0 (NTSC-U rev 0 is partly set up, and the README advises against working on it). Linker **MWCC GC/3.0a5.2**.
- README: "This will never be a decompilation of the full game… it certainly won't lead to ports." SDK/NW4R/MSL are out of scope. It's mainly for documentation and REL mods (https://github.com/SeekyCt/spm-rel-loader).

## Paper Mario: Color Splash (Wii U) / later games
- **No decomp found** on decomp.dev (223 projects) or in web search as of today. Wii U is PowerPC (Espresso), but no public tooling project for this game turned up.

## Recommended path
1. **Play/mod now:** PaperBoat with your own US PM64 ROM.
2. **Learn matching on finished code:** build pmret/papermario on this box (Linux supported; needs Rust/pigment64 and your US ROM) and study how functions map to C.
3. **Contribute:** TTYD. You need a **JP (G8MJ01) disc dump**. Use dtk + objdiff + decomp.me with MWCC GC/2.6, our Ghidra + powerpc binutils, and an AI draft → objdiff verify loop. Join the GC/Wii decomp Discord and claim a unit first.

## Developer profile: Paper Mario series
- **Developer:** Intelligent Systems developed Paper Mario (2000), TTYD (2004), and Super Paper Mario (2007); Nintendo published them. https://en.wikipedia.org/wiki/Paper_Mario_(video_game) , https://en.wikipedia.org/wiki/Paper_Mario:_The_Thousand-Year_Door , https://en.wikipedia.org/wiki/Super_Paper_Mario
- **Shared "EVT" scripting VM across games.** The PM64 decomp exposes `EvtScript`/`Evt` with script lists, priorities, map vars/flags, and child scripts (https://github.com/bates64/papermario-dx/blob/main/src/evt/script_list.c). TTYD has an event-script language with its own disassembler, *ttydasm* (https://github.com/PistonMiner/ttyd-tools). noclip.website ports the same evt logic between its PaperMario64 and PaperMarioTTYD renderers in one commit (https://seed.tty.garden/nmcdaniel/noclip.website/commit/b926de7c197597ff49fbf8c5b2e951a9d598ac69.patch). **Implication:** PM64's matched evt interpreter, opcodes, and API-function names are the best guide for TTYD's evt code, but expect ISA/compiler differences (MIPS GCC vs PPC MWCC) and new opcodes.
- **Existing TTYD symbol work:** jdaster64/ttyd-utils has near-complete symbol tables (us/eu/jp_symbols.csv), "including nearly all of the main binary's .text symbols" (credited to PistonMiner and Zephiles), plus scripts to dump sections/events/classes. It also includes a symbol diff of the JP demo vs US retail. https://github.com/jdaster64/ttyd-utils . **Check with the doldecomp/ttyd team how these names are used** before importing them.
- **SPM** credits TTYD docs (PistonMiner, Zephiles, Jdaster64, Jasper, NWPlayer123, Malleo, SolidifiedGaming, Diagamma) and the PM64 decomp in its README, so the lineage PM64 → TTYD → SPM is how the community actually works.
- **Toolchain habits:** N64 uses a GCC-family MIPS toolchain (custom pmret fork + gcc 2.7.2 + IDO 5.3 for some parts). TTYD uses MWCC GC/2.6 game code with GC/1.2.5n Dolphin SDK. SPM links with GC/3.0a5.2 (all from repos above).
