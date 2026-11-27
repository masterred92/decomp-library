# Lesson index

A list of individual rules in this library, including ones we later
replaced. Superseded entries stay here, with the date they changed, so
the history is visible. The narrative is in
[34](34-learning-timeline.md). Chapters
[01](01-history-and-tools.md)–[26](26-what-decomp-teaches-about-programming.md)
and [35](35-cross-project-decompiler-lessons.md)
are the decomp reference and are indexed in the [README](README.md).

## Current rules (late November 2026)

### Matching decomp and project setup
- Name edition and region differences as features (for example
  `#if FEATURE`, `WHEN`, `BY_LANGUAGE`); do not fork a source file
  per language. ([35](35-cross-project-decompiler-lessons.md) §1.1)
- Build tools should know a file format, never the game, and should
  be tested on invented fixtures that did not come from a ROM. Commit
  no ROM, blob, or extracted asset. ([35](35-cross-project-decompiler-lessons.md) §1.2,
  [08](08-legal-ethics.md))
- An empty function or object diff is not done until the full image
  hash matches for every edition you claim. A 100% objdiff translation
  unit still has to be flagged `Matching` and survive the link.
  Add a linking or shift test early, and keep intermediate objects for
  objdiff. ([35](35-cross-project-decompiler-lessons.md) §1.3)
- Do not replace a fake match with a worse kind: `signature` and
  `volatile` are the worst, `dead` the mildest. If honest C will not
  match, leave the function in assembly. ([35](35-cross-project-decompiler-lessons.md) §2.1,
  [25](25-golden-sun-matching-lessons.md) §3)
- Work only from the project you are contributing to, your own copy
  of the official image, and public documentation. Do not copy from a
  sibling clean-room tree or from leaked kits. Follow that project's
  AI policy. ([35](35-cross-project-decompiler-lessons.md) §3,
  [08](08-legal-ethics.md))
- A Thumb `pop {r0}` epilogue means `void`; `pop {r1}` means the
  function returns a value. Put a pooled constant into the destination
  register first (`x = 0x1ff; x &= field;`). Decrementing loop
  counters should be `int`, not a narrow type. ([35](35-cross-project-decompiler-lessons.md) §2.2,
  [25](25-golden-sun-matching-lessons.md) §2.9)
- A permuter or model score is not proof the C is correct; reread it
  for meaning and rewrite scripted wrappers as honest C before a PR.
  ([35](35-cross-project-decompiler-lessons.md) §2.3,
  [04](04-decomp-tooling.md), [05](05-ai-llm-decompilation.md))
- `asm("" : "+r"(x))` hides `x` from the optimiser; it is not a
  scheduling no-op. A leftover `volatile` on a pointer may come off
  while a plain temporary still has to stay. Fake matches that set
  register pressure with explicit clobbers are poor clean-fix
  candidates. ([35](35-cross-project-decompiler-lessons.md) §2.4)

### Shared libraries, release and review
- If a shared library's content changes, bump MINOR and add a new hash
  line; never rewrite a recorded hash. A check that only compares the
  current file to its current hash will miss two different bodies with
  the same MINOR. ([27](27-lessons-from-addon-releases.md) §1)
- After a `v*` tag, verify with `gh release view` and `gh run list`,
  then read the live store page. New CurseForge projects stay hidden
  until they are approved. ([27](27-lessons-from-addon-releases.md) §7,
  [33](33-release-ci-and-collaboration.md) §1)
- Do not weaken a shared-file CI check when the canonical repo is
  archived: unarchive, push one commit, re-archive, and confirm the
  checks are green. ([27](27-lessons-from-addon-releases.md) §6,
  [33](33-release-ci-and-collaboration.md) §2)
- Pin third-party actions that handle store credentials to a full
  commit SHA; keep internal machine paths out of shipped files.
  ([33](33-release-ci-and-collaboration.md) §2)
- A reviewer's pass is a code review of named commits, not a release
  approval. One writer per checkout; split work only when branches do
  not overlap. ([27](27-lessons-from-addon-releases.md) §10,
  [33](33-release-ci-and-collaboration.md) §3)
- Scan new files, commit messages and git history for personal
  information and credentials before a public push. Write commit
  messages with `git commit -F`.
  ([27](27-lessons-from-addon-releases.md) §9)

### Tests and Lua
- Prove a regression test by running it against the previous commit
  and confirming it fails there.
  ([27](27-lessons-from-addon-releases.md) §4,
  [28](28-tests-mocks-and-lua-pitfalls.md) §6)
- Do not use `and` / `or` as a ternary when a result can be false or
  nil. ([28](28-tests-mocks-and-lua-pitfalls.md) §1)
- Mocks must hide what the client hides, and must return nil for unset
  fields, not a do-nothing function.
  ([28](28-tests-mocks-and-lua-pitfalls.md) §2)
- Render from live state, not from a mockup.
  ([28](28-tests-mocks-and-lua-pitfalls.md) §3)
- A size threshold will not catch a clamped minimum; check that the
  result scales with quantity.
  ([28](28-tests-mocks-and-lua-pitfalls.md) §4,
  [32](32-auction-house-pitfalls.md) §1)
- Read automated-checker failures carefully; they can mis-parse
  ordinary source. ([28](28-tests-mocks-and-lua-pitfalls.md) §5)

### Hidden client data
- In combat, aura, cast and meter data are hidden by default, and the
  combat log is not available to addons. "Unknown" must stay unknown
  (`x or 1` is not a safe fallback). Re-check when combat ends.
  ([27](27-lessons-from-addon-releases.md) §2,
  [30](30-hidden-data-and-addon-apis.md) §1–§2)
- Choose a detection mode at runtime, with one source per spell.
  ([30](30-hidden-data-and-addon-apis.md) §3)
- Feature-test APIs that are not in the UI source. Keyboard-capturing
  overlays must not stay live into combat; do not edit the Escape-close
  list from a hide handler that is already walking it.
  ([30](30-hidden-data-and-addon-apis.md) §4–§5)
- A "Hide until I learn" choice is visible by default, saved per
  character, and held only for the session if the name is unreadable.
  ([27](27-lessons-from-addon-releases.md) §3)

### Data you can defend
- Test that controls change outputs. Cite the publisher who issued the
  figure. Check event timing and comparison direction. Totals must use
  the same estimates as their parts.
  ([29](29-honest-data.md) §1–§3)
- Regenerate the previous data release and byte-compare it before you
  trust a new diff. If a new build's tables match the last build, keep
  the old hotfixes. Do not invent numbers a hotfix never published.
  ([29](29-honest-data.md) §4–§5)

### Layout, contrast and the auction house
- Shorten the name first, not the figures; measure with the real font.
  ([27](27-lessons-from-addon-releases.md) §5,
  [31](31-ui-contrast-and-layout.md) §5)
- Move a colliding status to a free slot; hide it only when a
  combat-frozen height leaves no room. Size for the worst in-combat
  content before you freeze height.
  ([31](31-ui-contrast-and-layout.md) §3–§4)
- Check contrast over dark, snow and white, including additive glows
  and hover highlights. Judge icons at 64 and 32 pixels.
  ([31](31-ui-contrast-and-layout.md) §1–§2)
- Auction listing durations on this client are 2, 8 and 24 hours. Use
  the live deposit; keep a separate bucket for neutral houses; retry
  incomplete scan rows; bound what you store.
  ([32](32-auction-house-pitfalls.md))

## Superseded rules (kept for history)

- **Function counts in the first Golden Sun matching write-up.**
  An early count in [25](25-golden-sun-matching-lessons.md) was wrong
  and was corrected on **2026-10-14**; the scorecard was updated to 103
  functions on **2026-10-15**. Superseded by the scorecard now in
  chapter 25.
- **A per-spell aura lookup is usually readable in combat.**
  This was the first Gale Count combat fix on **2026-11-16**.
  Superseded the same day by: hiding is the default, and a lookup
  returns nothing unless the spell is marked never hidden. See
  [30](30-hidden-data-and-addon-apis.md).
- **A mock may feed plain aura data during a fight.**
  Used in those first Gale Count tests on **2026-11-16**.
  Superseded the same day by: mocks apply the client's hiding rules by
  default. See [28](28-tests-mocks-and-lua-pitfalls.md) §2.
- **Auction listing durations are 12, 24 and 48 hours.**
  Written in more than one addon note.
  Superseded **2026-11-20** by 2, 8 and 24 hours (found in a NeutralFlip
  comment and a Water Bill review). See
  [32](32-auction-house-pitfalls.md) §2.
- **A deposit that clears a size threshold is fine.**
  The old one-silver estimate sat above a small real deposit, so the
  threshold never fired.
  Superseded **2026-11-20** (NeutralFlip 0.3.3) by a quantity-scaling
  check. See [28](28-tests-mocks-and-lua-pitfalls.md) §4 and
  [32](32-auction-house-pitfalls.md) §1.
- **Conjured items always have a vendor price of zero.**
  An early Frost design note.
  Superseded **2026-11-20** by: read the item's sell-price field.
  See [29](29-honest-data.md) §6.
- **Hide the status if it touches the title.**
  First Frost layout rule on **2026-11-20**.
  Superseded later the same day by: move the status to the first free
  slot; hide it only when a combat-frozen height leaves no room. See
  [31](31-ui-contrast-and-layout.md) §3.
- **Figures can sit after a long name; the extra characters will
  ellipsis.**
  Implied by an early "we never cut the figures" comment.
  Superseded **2026-11-20** by a review finding: a trailing ellipsis
  hides whatever comes last, so shorten the name first. See
  [27](27-lessons-from-addon-releases.md) §5 and
  [31](31-ui-contrast-and-layout.md) §5.
- **If the current library hash matches the current file, the shared
  library is safe.**
  The usual lib check, still necessary, was not sufficient.
  Superseded **2026-11-20** (Frost 1.0.1 re-review) by: also bump MINOR
  and keep old hash lines immutable. See
  [27](27-lessons-from-addon-releases.md) §1.
- **Once store wording is fixed in the source, the live pages stay
  fixed.**
  After the **2026-11-13** listing edits.
  Superseded **2026-11-20** when some store summaries still carried a
  disclaimer that had already been removed from others. See
  [27](27-lessons-from-addon-releases.md) §8 and
  [33](33-release-ci-and-collaboration.md) §1.
