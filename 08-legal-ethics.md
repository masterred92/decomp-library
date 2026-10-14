# Legal & Ethical Notes (not legal advice; US-centric)
- **Copyright covers code and assets.** Decompiled code is arguably a derivative of the original. Projects avoid shipping assets/ROMs and require users to supply their own copy.
- **Interoperability case law:** *Sega v. Accolade* (9th Cir. 1992) held disassembly for interoperability can be fair use. https://en.wikipedia.org/wiki/Sega_v._Accolade . *Sony v. Connectix* (2000) had a similar result for emulation. https://en.wikipedia.org/wiki/Sony_Computer_Entertainment,_Inc._v._Connectix_Corp.
- **DMCA §1201** bans circumventing technical protection measures, with narrow exemptions renewed by the Copyright Office every 3 years. https://www.copyright.gov/1201/ . EU Software Directive Art. 6 permits decompilation for interoperability. https://eur-lex.europa.eu/eli/dir/2009/24/oj
- **EULAs** often forbid reverse engineering, which is contract risk.
- **Clean-room** practice (separate the people analysing from the people writing) lowers risk.
- Nintendo has taken down projects (e.g. DMCA notices) but has generally tolerated asset-free matching decomp repos. Don't count on that.
- Ethics: avoid cheats in online multiplayer, DRM cracking, and redistributing paid content. Credit communities.

## Expanded (2026-10-14): how the community keeps risk low, and what changed this year

> **Tutor's note.** This isn't legal advice. It explains *why* projects are set up the way they are, so the rules make sense.

### The standard "safe shape" of a decomp repo
| Rule | Why |
|---|---|
| No ROM, BIOS, disc image or extracted assets in the repo | Those are the copyrighted works themselves |
| You supply your own ROM, checked by SHA1 | The repo only holds new work plus a way to check it |
| Only short, factual descriptions of data (struct shapes, offsets) | Describing *how* data is laid out isn't copying the data (this is how [25 §4](25-golden-sun-matching-lessons.md) is written) |
| No circumvention of copy protection | DMCA §1201 ([copyright.gov/1201](https://www.copyright.gov/1201/)) |
| Quiet, non-commercial, credited | Takedowns tend to follow publicity, money or online-game impact |

### 2026: AI projects drew fast takedowns
* **Modern Warfare 2 (reported):** in early October, footage from Maurice Heumann's AI-agent decompilation was disabled after a
  copyright-owner report, and the code-host link went dark
  ([Kotaku](https://kotaku.com/activision-and-bethesda-crackdown-on-genai-mw2-and-fallout-projects-2000739580),
  [TheGamer](https://www.thegamer.com/ai-vibecoded-decompilations-takedown-fallout-mw2-dmca/)). His own write-up
  ([momo5502.com](https://momo5502.com/posts/2026-10-09-game-decompilation/)) says the code remains private.
* **Fallout recreation:** the same week, the creator reported a Bethesda cease-and-desist for an AI-built browser *recreation*. It wasn't a decomp.
* **What we can take from it (opinion, flagged):** speed and virality made these projects targets. They were recent, commercial and online-capable
  franchises, unlike decades-old handheld games. That doesn't make older games safe; Nintendo has also issued takedowns in the past.

### How this applies to our work
* Golden Sun PRs contain only our C, struct definitions and Makefile changes. The ROM stays on Kenny's PC, with a private copy on the box, and is never committed.
* This library contains no ROM bytes, original assembly dumps or assets.
* Addons ([WoW work](https://github.com/masterred92/wow-forever-addons)) are a separate case: they're governed by the game's ToS and only use Blizzard's public addon API.

### Common mistakes
* **"It's fine because I own the game."** Owning a copy lets you dump it for yourself. It doesn't let you share it.
* **Pasting original disassembly into blog posts or issues.** Describe it instead.
* **Treating a tolerated project as permission.** It isn't.
