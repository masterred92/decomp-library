# Reverse-Engineering Non-Game Desktop Apps (e.g. Adobe Photoshop)
## Recent news (Oct 2026). Treat it as developing; claims are attributed, not verified
- **PhotoCraft** (storytold/photocraft) calls itself "an open-source, clean-room reimplementation of Adobe Photoshop, rebuilt in pure Rust". Features: layers, masks, adjustment layers, type, vectors, brushes, PSD read/write, WASM. MIT/Apache-2.0. **Early alpha**, and its README says it's not a daily professional replacement. The README also says it was "implemented from public specs and observed behaviour only. No proprietary code, shaders or assets." https://github.com/storytold/photocraft
- Eric S. Raymond and others claimed online that it was produced by decompiling Photoshop into a spec and feeding it to an LLM. explainx.ai calls that **unproven speculation**, and the repo doesn't document it. https://explainx.ai/blog/photocraft-open-source-clean-room-photoshop-rust-esr-closed-source-dead-2026 , https://agihunt.info/en/p/1a11415432357b94a6f1702e2c3
- AGI Hunt reports a post by @kimmonismus about rebuilding 7 Adobe apps in Rust with an AI model, with PhotoCraft as the core repo. Attribution between that poster and the "ArtCraft Team" credited in the repo is unclear, so check the primary sources. https://agihunt.info/en/e/1a112832ae1f8ac5995c8fbebf0
- **OpenPhoto** (yuanzhixiang/openphoto) is a Rust editor that mimics Photoshop 2026's UI. It calibrates algorithms to Photoshop's *output* via probe images (black-box behavioural RE, not decompilation). It includes an MCP server. https://github.com/yuanzhixiang/openphoto
- Takeaway: the documented technique here is **black-box behavioural cloning + public specs + AI coding**. Decompilation is claimed but not shown.

## Techniques for large commercial C++ apps
- **Black-box first:** documented file formats, SDKs, scripting APIs, and output comparison (probe images, golden fixtures).
- **Static:** IDA/Ghidra/Binary Ninja. Recover C++ classes via RTTI and vtables (MSVC RTTI → class names/hierarchy). Use exported symbols, debug strings, and FLIRT/FunctionID signatures for statically linked libs (zlib, libpng, Boost, Qt). Ghidra C++ class analysis scripts: https://github.com/NationalSecurityAgency/ghidra (OOAnalyzer from CERT Pharos: https://github.com/cmu-sei/pharos)
- **Dynamic:** debuggers (x64dbg https://x64dbg.com/ , WinDbg), API tracing, Frida instrumentation https://frida.re/
- Focus on a subsystem (one filter, one parser), not the whole app. Modern apps are millions of LOC, so full decompilation isn't practical.
- Note that apps often have anti-tamper and licensing checks. Bypassing them raises DMCA §1201 issues.

## Plugin SDKs & file formats
- Adobe publishes the **Photoshop File Formats Specification (PSD/PSB)**: https://www.adobe.com/devnet-apps/photoshop/fileformatashtml/
- Photoshop plugin SDKs (C++ SDK and UXP) are at the Adobe Developer site: https://developer.adobe.com/photoshop/
- Open PSD implementations to learn from: GIMP's PSD plugin (https://gitlab.gnome.org/GNOME/gimp), psd-tools (Python) https://github.com/psd-tools/psd-tools , ag-psd (JS) https://github.com/Agamnentzar/ag-psd
- Format RE approach: make minimal diffs (save, change one thing, save, binary-diff), use the Kaitai Struct / 010 Editor templates, and fuzz round-trips.

## Legal side (not legal advice)
- **EULA:** Adobe's General Terms restrict reverse engineering except as permitted by law. Breaching that is a contract issue separate from copyright. https://www.adobe.com/legal/terms.html
- **DMCA §1201(f)** permits circumventing for interoperability of an independently created program, within limits. Triennial exemptions are at https://www.copyright.gov/1201/ . **EU Software Directive Art. 6** allows decompilation for interoperability only, not to make a substantially similar competing program. https://eur-lex.europa.eu/eli/dir/2009/24/oj
- **Clean-room:** team A writes a spec, team B (which never saw the code) implements it. Historic example: Phoenix's IBM BIOS clone. https://en.wikipedia.org/wiki/Clean_room_design . *Oracle v. Google* (2021) held API reimplementation was fair use in that case. https://en.wikipedia.org/wiki/Google_LLC_v._Oracle_America,_Inc.
- An LLM "clean room" is legally untested: whether a model that saw decompiled code can serve as team B is an open question. Also note that trademarks (the "Photoshop" name, UI icons) are a separate issue.
