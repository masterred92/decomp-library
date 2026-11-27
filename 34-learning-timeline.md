# 34: Learning timeline (October–November 2026)

Written 2026-11-20. A chronological companion to this library, so you
can see how the lessons accumulated rather than only how they stand
now. Dates below come from this repo's git history, public addon tags,
and dated review notes. They are not a complete diary.

The short "how this evolved" section at the end of
[27](27-lessons-from-addon-releases.md)–[33](33-release-ci-and-collaboration.md)
tells the same stories in place. Individual rules that we later
replaced are kept in the [lesson index](INDEX.md).

## 2026-10-07 — the library begins

The first commit of this repo set out a decompilation reference:
history and tools, matching projects, static recompilation, tooling,
AI-assisted decomp, platforms, workflow, legal notes, and a shortlist
of games. The same day added Golden Sun, Zelda, Dragon Ball Z on GBA,
a Harry Potter GBC study, developer approaches, learning paths,
language history, a tour of unusual websites, and a Linux and Git
chapter ([01](01-history-and-tools.md)–[24](24-linux-and-torvalds.md),
[16](16-golden-sun.md)–[20](20-developer-approaches.md)).

## 2026-10-14 to 2026-10-15 — matching, then general programming

A scene update and a first Golden Sun matching chapter recorded
patterns from real pull requests. Function counts in that chapter were
corrected the same day, and by 15 October the scorecard stood at 103
functions (PRs #32–#54). A second chapter drew out the general
programming lessons: compilers, data layout, types, debugging,
verification, automation and collaboration
([25](25-golden-sun-matching-lessons.md),
[26](26-what-decomp-teaches-about-programming.md)).

## 2026-10-14 to 2026-10-30 — first auction addons

On 14 October a MatTooltip 0.1.0 landed in the shared wow-forever-addons
repo; on 18 October PriceLog 0.1.0, FarmTracker 0.1.0 and MatTooltip
0.2.0 were tagged. A written review on 25 October listed fee, deposit,
neutral-house and scan problems that later became
[32](32-auction-house-pitfalls.md). NeutralFlip 0.1.0 followed on
26 October. A 0.3.0 release on 30 October shared prices, scans
and a self-check across those addons.

## 2026-11-03 to 2026-11-09 — "looks right" is not enough

The Policy Simulator repository opened on 3 November, and the first
correction was that controls looked right but did not change outputs.
Drafts over the next few days mis-cited sources, got event timing
wrong, and once reversed a rate comparison; a 9 November total did not
match its parts. Review 9 and Review 10 landed that day. Those notes
became [29](29-honest-data.md). WoW: Forever launched on 4 November
(Blizzard).

## 2026-11-10 to 2026-11-13 — data rebuilds and store wording

Dwarf Explorer's Bible 1.0.5 (10 November) rebuilt recipe data against
a new client build and recorded that the output was unchanged.
Version 1.1.0 on 12 November applied official hotfixes to that data.
Gale Count, Reckoning Checkoning and related trackers reached 1.0.0
the same day. On 13 November, store summaries were edited to drop a
disclaimer we had decided not to use.

## 2026-11-16 — hidden combat data, and a reviewer's first blockers

Gale Count 1.0.2 shipped after a first combat fix and its tests were
found to assume readable aura data in a fight. Hiding is the default;
mocks have to do the same; detection is chosen at runtime
([30](30-hidden-data-and-addon-apis.md),
[28](28-tests-mocks-and-lua-pitfalls.md)). The same day, a simulator
refresh review caught a cap applied in only one mode, and UI mockup
work began on contrast against a bright world
([31](31-ui-contrast-and-layout.md)).

## 2026-11-20 — a full day of release lessons

Several public tags landed this day: Frost suite 1.0.0 and 1.0.1
(seven addons), NeutralFlip 0.3.3, Reckoning Checkoning 1.0.1, and
Dwarf Explorer's Bible 1.2.0.

From the Frost work: bump a shared library's MINOR and never rewrite a
recorded hash; re-check spell knowledge when combat ends; a
"Hide until I learn" state that is safe when a name is unreadable;
shorten the name, not the figures; unarchive a canonical repo to fix
CI rather than weaken the check ([27](27-lessons-from-addon-releases.md)).
A layout rule that hid a colliding status was replaced later the same
day by moving the status instead ([31](31-ui-contrast-and-layout.md)).

From NeutralFlip and Water Bill: listing durations are 2 / 8 / 24
hours, and a deposit check has to scale with quantity
([32](32-auction-house-pitfalls.md),
[28](28-tests-mocks-and-lua-pitfalls.md)).

From Dwarf Explorer's Bible 1.2.0: regenerate the previous release and
byte-compare it; a new client build can arrive before its hotfix list;
do not invent numbers a hotfix never published
([29](29-honest-data.md)). Lua `and` / `or` pitfalls and truthful
mocks were confirmed the same afternoon
([28](28-tests-mocks-and-lua-pitfalls.md)).

From the stores and CI: re-read live listings after a release (some
pages still had a disclaimer that had been removed from others on
13 November); packager uploads appeared in about eleven to eighteen
minutes; pin credential-handling actions to a SHA
([33](33-release-ci-and-collaboration.md)). A public-repo hygiene pass
the same day is summarised in [27 §9](27-lessons-from-addon-releases.md).
This library gained chapter 27, then chapters 28–34.

## 2026-11-27 — cross-project decompiler lessons

After a week of Golden Sun matching (including
[Coaltergeist/goldensun-decomp#102](https://github.com/Coaltergeist/goldensun-decomp/pull/102))
and a read-only look at
[PascalPixel/alchemy](https://github.com/PascalPixel/alchemy)
(both Golden Sun games, all twelve editions, already 100% matching),
this library gained
[35](35-cross-project-decompiler-lessons.md). That chapter records
how 100% projects structure multi-edition builds and asset tools,
Alchemy's `FAKEMATCH` severity ladder, categorized assembly-to-C
patterns for humans and ML-assisted matchers, a playbook of project
practices from Alchemy, zeldaret, pret, doldecomp and objdiff, and a
checklist of what our own decompiler should check, emit and verify.
A local Golden Sun 2 fake-match pass the same day improved 1 of 10
targets (a leftover `volatile` came off a pointer; the temporary
itself had to stay). The other nine were the hard residue: empty
`asm` barriers that hide values from `-Os`, signature and clobber
tricks that pin registers, and temporaries coupled to other
phrasing. Those results are in [35 §2.4](35-cross-project-decompiler-lessons.md).
