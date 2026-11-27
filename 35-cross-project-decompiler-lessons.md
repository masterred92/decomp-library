# 35: Cross-project lessons for a game decompiler

Written 2026-11-27. A practical lesson for our own decompiler and
matching workflow, drawn from projects that already match (or nearly
match) a whole game: [PascalPixel/alchemy](https://github.com/PascalPixel/alchemy)
(Golden Sun 1 and 2, all twelve regional editions), the
[zeldaret](https://github.com/zeldaret) family, [pret](https://github.com/pret),
[doldecomp](https://github.com/doldecomp), and
[encounter/objdiff](https://github.com/encounter/objdiff). Background
on those projects is in [02](02-matching-decomp-projects.md),
[16](16-golden-sun.md) and [17](17-zelda.md). The day-to-day matching
loop is in [07](07-workflow.md) and [04](04-decomp-tooling.md). Our
own gcc 2.96 / Thumb habits from
[Coaltergeist/goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp)
PRs are in [25](25-golden-sun-matching-lessons.md) and
[26](26-what-decomp-teaches-about-programming.md).

> **Tutor's note.** "Matching" still means: write C that the *original*
> compiler turns back into the *exact* bytes of the official image.
> This chapter is about *how those projects are set up* and *how the
> compiler was persuaded*, so a human or an ML-assisted decompiler can
> reuse the patterns. Everything is described at the C and convention
> level. There are no ROM bytes, no dumps of original assembly from a
> game, no assets, and no game text. Illustrative Thumb snippets are
> generic instruction patterns, not extracts from a cartridge.
> Alchemy was first inspected read-only. A later local pass then
> tried to clean high-impact Golden Sun 2 fake matches; the results
> are in §2.4. The inventory and severity ladder in §2.1 come from
> `CONTRIBUTING.md` and a count of `FAKEMATCH(` marks.

[Chapter 16](16-golden-sun.md) noted that Alchemy was closed to
outside contributions until both games were done. As of this writing
both games are fully rebuilt in all six languages, and
[CONTRIBUTING.md](https://github.com/PascalPixel/alchemy/blob/main/CONTRIBUTING.md)
welcomes pull requests for fake-match fixes, names, comments, docs,
assets and the build. Alchemy's clean-room rule is unchanged: work
only from your own ROMs, that repository, and public documentation —
not from any other Golden Sun project, and not from leaked code or
development kits. That rule is why this chapter learns *practices*,
not lines of their game C.

---

## 1. Project and build architecture

### 1.1 Multi-edition builds, named from day one

Alchemy builds twelve editions from one tree: two games (`tbs`, `tla`)
times six languages (`ja`, `en`, `de`, `es`, `fr`, `it`). `make`
builds all twelve and checks each against a hash in `rom.sha1`.
`make tbs-en` (or `make compare-tla-de`) builds one edition.

Edition differences are *named*, not copied into twelve files:

- A behavioural difference is a feature, set per edition in
  `include/edition.h`, and asked for with `#if FEATURE` or
  `WHEN(FEATURE, on, off)`.
- Text and layout that follow the language use `JAPANESE_OR(ja,
  others)`, the other `<LANGUAGE>_OR` pickers, or
  `BY_LANGUAGE(ja, en, de, es, fr, it)`.
- A repair of an original bug or of undefined behaviour is a `FIX_`
  flag, off in every matching edition, and turned on only behind
  `BUGFIX` / `UBFIX`.

zeldaret/oot does the same job with *named versions* (NTSC 1.0–1.2,
PAL, GameCube retail and Master Quest, debug builds) and a default
version; pret publishes the expected SHA-1 (and sometimes a Rev 0 /
Rev 1 pair) in the README. The shared lesson is: **name the
difference**. Forking a source file per region makes every later fix
twelve times as hard.

### 1.2 Asset pipelines: structured inputs, format-only tools

Alchemy's contributing rule is blunt: no raw dumps and no blobs.

| Kind | Source form |
|---|---|
| Picture | Indexed PNG with the real palette, drawn the way the game shows it |
| Sound | WAV (samples) and MIDI (songs) |
| Text | `.po` catalogs under `text/` |
| Table | `.tsv`: a header line, one row per thing, decimal numbers |

A build tool is a small C program in `tools/` (`ags-graphics`,
`ags-audio`, `ags-music`, `ags-text`, `ags-maps`, `ags-overlays`,
`ags-header`, and so on). It knows a *file format*, never this game.
The tool tests (`tools/tests/tests.c`) exercise those programs with
**small invented fixtures**, not a ROM: editable inputs, encoded
layout, dependency updates, and rejected inputs that must fail cleanly
(exit status 1) and leave an existing output file intact.

`make test` also runs `ags-check` over every tracked file under
`games/` and `tools/`. The check is a project standard, not a
linter-of-taste: allowed kinds only (C, headers, honest hand-shaped
assembly, `.tsv`, `.png`, `.wav`, `.mid`, `.po`, linker scripts,
Makefiles, `rom.sha1`, and a folder `README.md`); no `.bin` or `.gba`;
a scene or resource's number is a **linker symbol**, never a bare
integer in a table. File length is an *alarm* (looks like two things
in one file), not a hard split.

zeldaret projects extract assets from a user-supplied ROM (ZAPD on
the N64 titles) and say plainly in the README that the repo does not
ship the assets needed to build. pret is the same shape. That is the
[safe repo shape](08-legal-ethics.md): the official image stays with
the person who owns a copy; the repo holds tools, C, and hashes.

**For a new project:** invent the formats and the tools first, test
them on fixtures that never came from a cartridge, and only then point
them at a user-supplied image.

### 1.3 The verification ladder

Successful projects do not treat a single empty diff as the end of
the work. They stack checks:

1. **Instruction / function.** [asm-differ](https://github.com/simonlindholm/asm-differ),
   [decomp.me](https://decomp.me/), or
   [objdiff](https://github.com/encounter/objdiff) compares your object
   against the target. objdiff reads two relocatable `.o` files, can
   rebuild on a source change, and powers [decomp.dev](https://decomp.dev)
   progress. It supports ARM (GBA, DS, 3DS), ARM64, MIPS, PowerPC and
   others. Generate `objdiff.json` from `configure.py` and keep it out
   of git (objdiff's own README recommends this).
2. **Translation unit.** doldecomp/mkdd is explicit: a TU at 100% in
   objdiff is **not done** until it is flagged `Matching` in
   `configure.py` *and* `ninja` links a DOL whose SHA-1 matches
   `build.sha1`. Statuses are `Matching` (your object is linked),
   `NonMatching` (the extracted object is linked), and `Equivalent`
   (same behaviour, only used in a `--non-matching` build). Melee,
   now 100% and shiftable, uses the same configure/ninja/objdiff loop.
3. **Whole image.** Alchemy's `make` / `make compare-editions` must
   print OK for all twelve hashes. Our Golden Sun PRs still run
   `make compare` on the main ROM and every overlay
   ([25 §5](25-golden-sun-matching-lessons.md)). Section padding can
   break a ROM after every function already "matches" ([25 §1](25-golden-sun-matching-lessons.md)).
4. **Tools and style.** `make test` (tool tests, `clang-format`,
   `ags-check`). pret/pokeplatinum and pret/pokeheartgold run
   `clang-format` from a hook or `make format`, and CI rejects an
   unformatted PR.
5. **Relocatability.** Matching a hash is not the same as being able
   to add a row. Alchemy's `make test-linking` rebuilds one edition
   with an extra named row in the middle of the scene, resource and
   map lists, then checks that every earlier id is unchanged, every
   later id has moved by one, and every overlay still points at its
   own image. `make SHIFT=n` moves everything after the header
   (and is *not* compared to `rom.sha1`). zeldaret READMEs still
   warn that some regions may not be shiftable; doldecomp/melee
   advertises that its DOL *can* be shifted. Build this test in
   early, while names are cheap to add.
6. **Modern compile (optional).** Alchemy's `make modern` sends the
   same C through a current GCC. It will not match the ROM. It will
   tell you whether the C is still C.

Keep the intermediate `.o` and ELF files (Alchemy's Makefile says it
follows pret here). objdiff and a size report need them.

---

## 2. Compiler matching and the fake-match hierarchy

### 2.1 Alchemy's `FAKEMATCH` severity ladder

A fake match is code bent into an odd shape so the old compiler
emits the original instructions. Alchemy marks each one with the
plain form it *should* have, then asks contributors to solve it
without replacing it by a *worse* kind:

> `signature` and `volatile` are the worst, `dead` the mildest.

`volatile` is in that policy; a read-only inventory on 2026-11-27
found no `FAKEMATCH(volatile)` marks in the tree. Assembly is allowed
only in `.s` files and in the `ASM_` macros of `include/system/asm.h`.

Counts in `games/tla/` (The Lost Age), 184 marks:

| Kind | Count | What it usually is | How you improve it |
|---|---|---|---|
| `signature` | 1 | A parameter or local whose *type* is a lie (for example a pointer that actually holds a size) so the compiler picks a convenient representation | Give it the real type and rephrase the users |
| `volatile` (policy; unused here) | 0 | A `volatile` qualifier used only to stop a load or store being combined or moved | Rewrite lifetimes and order until the qualifier can go |
| `asm` | 17 | An instruction C cannot say: a BIOS stop, a particular `stmia` / `ldmia` shape, `smull` with a chosen RdLo/RdHi pair, a local pinned to a register | Keep it isolated in a named macro or a `.s` file; do not sprinkle inline asm into ordinary C |
| `barrier` | 5 | An empty `asm` that freezes scheduling or a literal-pool break, or that "rewrites" a value the C already holds | Delete the barrier and change declaration order, types, or control flow until the pool and schedule land naturally |
| `wrapper` | 2 | An extra inline layer (sometimes edition-specific) around a call or address | Inline the plain call once the compiler no longer needs the layer |
| `temporary` | 48 | An extra local, pointer alias, or copy (`sprite` holds `&cursorSprite`; bits held in a local instead of `*flags |= bits`) | Use the value at the point of use; try the order of declarations and the type of the local first |
| `dead` | 111 | A store nothing reads, two updates that cancel, a test whose arms are alike, an empty once-only block, a value written as XOR instead of a compare | Delete the dead work, or keep a *milder* residue if that is all that still matches |

The same kinds appear in `games/tbs/` (80 marks; no `signature` or
`wrapper` there). To solve one: write the plain form, run one edition
(`make compare-tla-en`), and only then `make` for all twelve. If it
breaks, try declaration order, a local's type, or a flipped `if`
before reaching for a worse mark.

### 2.2 Categorized assembly-to-C patterns

These are the translations that showed up again and again in our
Thumb / gcc 2.96 matching ([25](25-golden-sun-matching-lessons.md)
and the week of 2026-11-20 to 2026-11-27) and in the fake-match
kinds above. Each row is written so a person or a model can study
it: the assembly *pattern*, the C you write first, the C that
matches, and *why* the compiler emitted it. The assembly is
illustrative, not a dump from a game.

#### Control flow and loops

**Counted `for` with an early `break`, not an equivalent `while`.**
Old GCC keeps the induction variable and the exit test in the shape
you wrote. A wait of at most 300 frames matched as:

```c
for (i = 0; i < 300; i++) {
    if (state[1] <= 0xff && state[2] <= 0xff)
        break;
    WaitFrames(1);
}
```

A `while` that means the same thing did not
([25 §2.13](25-golden-sun-matching-lessons.md)).

**Put the length check first in a combined condition.**
`for (n = 0; n < 24 && list[n] != 0; n++)` and the swapped `&&`
compile two different branch orders, the same way checking `< 0`
before `> max` in a clamp sets branch order ([25 §2.8](25-golden-sun-matching-lessons.md)).

**Decrementing counters should be `int`, not `u8`.** A narrow
counter forces extra masks or zero-extends on every step. The
original usually counted in a 32-bit register.

**`z - 1` as an expression, not `z--`.** Passing `f(x, z - 1, …, z - 1)`
keeps the old value alive; `z--; f(x, z, …)` makes the compiler
allocate a register for the new value and swapped r5/r6 across a
family of functions ([25 §2.14](25-golden-sun-matching-lessons.md)).

**Index a table of pointers; walk a pointer through one block.**
`cursor = cursors[i]` matched for a table of pointers;
`p += n; *p` matched when stepping through fields of one struct.
Ask which kind of data you are looking at ([25 §2.4, §2.10](25-golden-sun-matching-lessons.md)).

#### Register allocation and local reuse

**Reuse one local for two finished values** when the target reuses a
register. Giving each value its own name can keep both alive and
flip the colouring (seen on `Anim_Attack` in the 2026-11-27 batch).

**Load into named locals first** when the original does
(`x >>= 20; z >>= 20;` at the top; a separate `base`, `ofs`, then
pointer). Extra locals are a register-allocation tool, not just
readability ([25 §2.5, §2.6](25-golden-sun-matching-lessons.md)).

**Do not save what the original re-reads.** If the assembly loads
`u->numDjinn[element]` twice, write it twice. Caching it in a local
was a one-`mov` miss until we stopped ([25 §2.11](25-golden-sun-matching-lessons.md)).

**Keep a tested-and-passed value in a named local** when the
assembly uses the value that already survived a compare, rather than
loading the table again for the call ([25 §2.12](25-golden-sun-matching-lessons.md)).

**Cache an IWRAM address in a high register** the way the original
does: `clock = &iwram_var; *clock;` not a fresh `&iwram_var` on
every access. The address lives in the register across the body.

#### Literal pools and bitmasks

Thumb immediates are small, so large constants and addresses sit in
a *literal pool* beside the function and are loaded from there
([26 §1](26-what-decomp-teaches-about-programming.md)).

**Put the pooled constant in the destination register first.**

Naive C (often an extra `mov` after the `and`):

```c
item = field & 0x1ff;
```

Matching C (the pool load lands in the register the `and` already
uses):

```c
item = 0x1ff;
item &= field;
```

**Mask into an `int` temporary** rather than a narrow type, or the
compiler inserts an extend or an extra `mov` ([25 §2.7](25-golden-sun-matching-lessons.md)).

**An empty `asm` barrier** (Alchemy `FAKEMATCH(barrier)`) is often
there only to force a pool break after a menu or to pin a value
across a scheduled region. Prefer declaration order and control-flow
shape; a barrier is a last resort, and a worse one than a temporary.

#### Struct layout and pointer arithmetic

**Structs beat raw offsets and masks.** Load size, signedness and
address maths come from the types. Psynergy slots that are 4 bytes
apart but read as halfwords matched as
`struct UnitMove { u16 id; u16 __unk2; }`, not as the same maths on
a byte pointer ([25 §2.1](25-golden-sun-matching-lessons.md)).

**Signedness is visible in the load.** `ldrsh` means `s16`; `ldrh`
means `u16`. Believe the instruction ([25 §2.2](25-golden-sun-matching-lessons.md)).

**Write address maths in the order the original used.**

Naive (same value, different instructions):

```c
return tiles[x + (z << 7)].unk2;
```

Matching:

```c
tiles += x + (z << 7);
return tiles->unk2;
```

**Compute `ofs` first, then `base + ofs`**, to get a register-offset
load (`[r4, r3]`) rather than a folded immediate or a different
scratch.

**Recompute from the same base** (`base->member[i]`) instead of
chaining offsets from a previous pointer, once the struct view has
array or struct members ([25 §2.14](25-golden-sun-matching-lessons.md)).

**The write order of expressions is the instruction order** on gcc
2.96: `total - f(x) + 15` is not `total + 15 - f(x)`.

#### Function signatures and epilogues

**The Thumb return-type tell.** A function that saved `lr` returns
by popping it into a free low register and branching:

- `pop {r0}` then `bx r0` — r0 is free — the function is `void`.
- `pop {r1}` then `bx r1` — r0 is taken — the function returns a
  value.

Some functions end with `pop {r1}` and never write a `return`
expression; they matched as implicit-`int` with no `return`
statement ([25 §2.9](25-golden-sun-matching-lessons.md)). Copying
the already-matched signature of a sibling in the same file keeps
that consistent.

**`FAKEMATCH(signature)`** is the nuclear version of this: the
declared type is wrong on purpose (a `Uint32 *` that actually holds
a size). Solve it by giving the value its real type, even if that
means a local with a wider or narrower view at the use.

An unprototyped `extern void f();` when `f` is defined later with
parameters is old C, and it changes argument setup
([25 §2.14](25-golden-sun-matching-lessons.md)).

#### Compiler barriers and fake matches

Work down the ladder in §2.1, never up:

- Prefer a `dead` residue (a store nothing reads, a cancelling
  update, an empty once-only block) over a new `temporary`.
- Prefer a `temporary` (an alias, a copy, bits in a local) over a
  `wrapper` or a `barrier`.
- Isolate true non-C sequences as `asm` macros, one purpose each.
- Do not add `signature` or `volatile` to "get the bytes". If honest
  C will not match, leave the function in assembly — the policy we
  used on goldensun-decomp ([25 §3](25-golden-sun-matching-lessons.md)).

`decomp-permuter` is part of this rung, not a replacement for it.
It scores *bytes*, not meaning. On `Func_8015e8c` its best result
after five minutes compiled closer and did something different; we
threw it out ([25 §3](25-golden-sun-matching-lessons.md),
[04](04-decomp-tooling.md)). Wrappers such as `if (1)` or
`do { } while (0)` that a permuter or a script emits must be
rewritten as honest C before a PR, or not used.

### 2.3 A short matching loop (for humans and models)

Draft (structs first) → compile with the *exact* historical compiler
and flags → diff at the object → nudge phrasing (locals, order,
types, loop shape) → reread for meaning → `make compare` (or `ninja`
plus the image hash) on every edition you claim. The permuter is
last, and its output is a suggestion you have to be able to defend.

### 2.4 A live GS2 fake-match pass (2026-11-27)

A local cleanup pass on The Lost Age tried ten high-impact marks.
All twelve editions still compared OK and `make test` passed. The
honest score was **one partial improvement out of ten targets**.
That is not a poor effort so much as a measure of what is left:
the remaining Alchemy fake matches are the hard residue, not a
backlog of easy rewrites.

The one improvement dropped a leftover `volatile` from a pointer
used to read a scroll value. The `FAKEMATCH(temporary)` itself had
to stay. Writing the global into a local first, as if it were an
ordinary load, delayed the Thumb `ldr` / `mov` into a saved
register until after a later row was read. A plain pointer
temporary (without `volatile`) kept the original order:

```c
/* leftover volatile was not required */
int *p = &scroll;   /* load of &scroll happens early; use *p */
/* not:  x = scroll;  then a later use, which can delay the load */
```

The other nine were skipped after two or three clean attempts each:

- A `signature` that stores a constant through a `Uint32 *` kept
  that constant in a saved register (the one that later holds an
  allocation). Clean types put it in a scratch register instead.
- An empty `asm` barrier around a loop counter hid from `-Os` that
  the counter starts at zero, which preserved a compare on loop
  entry. Without the barrier, gcc rewrote the loop as a do-while
  and the registers changed. A near-miss on a second barrier only
  swapped a stack-address `add` and a `mov` of a return value
  after a call.
- A barrier that listed explicit clobbers plus an uninitialised
  `asm` output was a poor candidate: it was forcing register
  pressure, not merely freezing a schedule.
- An inline `wrapper` whose return temporary pinned allocation
  shifted literal-pool labels by two instructions when inlined
  away.
- Several `temporary` marks were coupled to other phrasing
  (two-step versus combined expressions, a dead store that still
  paid for a load, or taking the address of a local early) and
  could not be removed on their own.

Five takeaways for the matcher, human or automated:

1. The July 2000 GCC is very sensitive to which value lives in a
   long-lived saved register, whether a loop counter is hidden
   from the optimiser, `x = c; x &= y` versus `x = c & y`, and
   when a local's address is taken.
2. `asm("" : "+r"(x))` is not a scheduling no-op. It hides the
   value of `x` from the optimiser, which can be the whole reason
   a loop or a compare still matches.
3. `volatile` on a pointer can be a leftover from an earlier
   attempt. Try a plain temporary first; it may already be enough.
4. Fake matches that combine explicit clobbers with an
   uninitialised `asm` output are directly manufacturing register
   pressure. They are poor clean-fix candidates.
5. A fast inner loop for this work: build one unity `.o`, diff
   only the function against a saved baseline, and run the full
   `make` once a candidate is actually close.

---

## 3. What makes future decomp projects easier

These are project-level practices, not function-level tricks. A new
matching decomp that starts with them will scale; one that skips
them will spend a year retrofitting.

### Provenance and the safe shape

- **User-supplied official image, hash in the repo, no assets or
  binaries committed.** zeldaret, pret, doldecomp and Alchemy all
  say this in the README. See [08](08-legal-ethics.md).
- **A written clean-room rule.** Alchemy: only this repository,
  public documentation, and your own ROMs; no other Golden Sun
  project; no leaked code or kits. zeldaret/mm: contributors must
  never have accessed leaked N64 SDK, iQue, libultra, or sibling
  game source. Write the rule down before the first PR.
- **A written AI rule, because projects disagree.**
  pret/pokeheartgold prohibits LLM-generated contributions (with a
  narrow, disclosed exception for purely mechanical chores).
  doldecomp/mkdd rejects PRs where an LLM decompiled whole functions
  or matched a whole TU; docs and a hint on the last few diffs are
  allowed, and it asks you to try Discord and decomp.me first.
  Our own stance for *this library* is in
  [05](05-ai-llm-decompilation.md): the compiler-plus-hash is the
  oracle, not the model's confidence. Follow the project you are
  posting to.

### Build and tooling, from the first commit

- **Multi-edition feature flags** (§1.1), not per-region file
  copies.
- **Generic, format-only tools with tests on invented fixtures**
  (§1.2). The tools must not know the game's story, maps or
  names.
- **Generate `objdiff.json` from the build script**; hook
  [decomp-permuter](https://github.com/simonlindholm/decomp-permuter)
  and decomp.me as soon as a compiler image exists.
- **A three-state object flag** (`Matching` / `NonMatching` /
  `Equivalent`) in `configure.py`, the way doldecomp does, so a
  half-finished TU cannot silently enter the linked image.
- **`make compare` (or `ninja` + SHA-1) in CI**, plus a format
  check. Alchemy also runs `make test` (tools + `ags-check`) on
  every push.
- **A linking or shift test from day one** (§1.3). Names for list
  rows are cheap early and painful to add after bare numbers have
  spread.
- **Keep intermediates.** objdiff compares `.o` files; a "clean"
  build that deletes them makes the main UI of the project useless.
- **Pin the historical compiler and flags in the repo** (submodule
  or `tools/`), and offer a `modern` target that will not match.

### Source conventions that keep a tree readable at 100%

- **One mechanical format.** Alchemy uses clang-format 20 with the
  repo's `.clang-format` (four-space indents, braces on their own
  lines, aligned declarations). pret/pokeplatinum and
  pokeheartgold do the same with hooks so reviewers do not have to
  nibble whitespace. Disable format only around compiler-specific
  inline asm (heartgold's `// clang-format off` around
  `NONMATCHING`).
- **Docs live next to the code.** Alchemy: a `README.md` in each
  folder, longer pages in `docs/`, and the site built from both;
  a change to behaviour updates the page in the same PR.
  zeldaret/oot and mm ship a decomp tutorial, a style guide and
  review guidelines. "Matching" without names and docs is not
  finished — leftover `func_08049CD4` symbols at 100% still need
  work ([17](17-zelda.md)).
- **A tagged leftover vocabulary.** Alchemy's `FAKEMATCH(kind)`,
  `XXX:` (an open question plus what would settle it), `BUG:` /
  `UB:` (kept, with a fix behind a flag), `DEAD:` and `Unused:`.
  grep-able marks are a contribution queue. zeldaret and pret use
  `NON_MATCHING` / `NON_EQUIVALENT` (or `NONMATCHING`) around a
  `GLOBAL_ASM` / extracted object so a partial file still builds.
- **Do not throw handwritten assembly into the decomp queue.**
  zeldaret/tmc's contributing guide tells you to ignore the
  "handwritten assembly" section of the linker script.
- **Small, claimed units of work.** mkdd: one translation unit per
  PR, taken to 100%, draft PR opened early so others do not
  collide. Our Golden Sun batches: a few functions, themed, no
  overlapping header changes ([25 §5](25-golden-sun-matching-lessons.md)).
  Start with small leaf functions or self-contained actors
  (zeldaret/mm's first-contribution advice; [07](07-workflow.md)).

---

## 4. Design takeaways for our own game decompiler

What an automated or assisted decompiler should **check**, **emit**
and **verify**. This is the training and product checklist; the
patterns in §2 are the features.

### Check (before it proposes C)

- The **exact compiler and flags** (Camelot gcc 2.96 with
  `-fcall-used-r4` is not agbcc, and is not a modern Clang). A
  near-match with the wrong flags can never become exact
  ([04](04-decomp-tooling.md), [16](16-golden-sun.md)).
- Load **size and signedness** (`ldrsh` / `ldrh` / `ldrb`).
- The **epilogue register** (`pop {r0}` vs `pop {r1}`) before
  guessing `void`.
- **Loop shape**: counted `for` vs `while`, length-check order,
  `int` vs narrow counters, expression vs decrement.
- **Literal-pool placement**: which register the pool load lands
  in, and whether a constant should be written to the destination
  first.
- **Address form**: struct field vs raw offset; `ofs` then
  `base + ofs`; cached address in a high register; index vs
  pointer walk.
- **Lifetimes**: reused registers vs re-read fields vs a value
  kept from a compare.
- **Call-clobbered registers** under the project's flags (with
  `-fcall-used-r4`, a live-across-call value is not in r4).

### Emit

- **Plausible period C** (C89 plus the compiler's GNU extensions,
  or the project's C++ dialect), structured with structs, enums
  and named feature flags — not a soup of casts.
- **Honest code first.** No new `signature` or `volatile`
  fakematches; no permuter wrappers. If it cannot match honestly,
  emit a clearly marked `NON_MATCHING` / leftover `.s`, or a
  `FAKEMATCH(kind)` at the *mildest* kind that still matches, with
  a comment stating the plain form.
- **Minimal diffs**: one function or one TU, no unrelated
  reformatting (the formatter should already have run).
- **Structured assets and names**, never blobs or bare list
  indices.
- **A short comment when the phrasing is non-obvious** (why the
  local exists, why the condition is written this way).

### Verify

- Object-level diff empty (objdiff / asm-differ / decomp.me).
- Full image hash, **every edition you claim**, including overlays
  or RELs.
- Tool tests and format checks.
- Linking or shift test when the change touches ids, tables or
  layout.
- A **semantic reread** of any permuter or model output. Bytes are
  not meaning ([05](05-ai-llm-decompilation.md)).
- **Clean-room discipline:** no copying from a sibling project
  (Alchemy vs goldensun-decomp is the worked example); no leaked
  SDKs; follow the target repo's AI policy.
- Combined merge of open branches still matches, so PRs do not
  fight each other ([25 §5](25-golden-sun-matching-lessons.md)).

### Training notes for an ML-assisted matcher

- Organise examples in the categories of §2, each with naive C,
  matching C, and the compiler reason. Reward the matching
  phrasing; penalise a worse fakematch even if the bytes agree.
- The label is the **compiler oracle** (rebuild + hash / objdiff),
  never "the model is sure".
- Do not train on leaked source, on a project's C copied into
  another clean-room tree, or on ROM bytes.
- Prefer projects that already tag leftovers (`FAKEMATCH`,
  `NON_MATCHING`, `XXX:`) — those tags are negative and partial
  labels.
- A 100% match is not a finished dataset: names, docs and
  shiftability are separate targets.

---

## Common mistakes

- **Treating % matched as % done.** Naming, documentation and
  shiftability come after the hash is green ([02](02-matching-decomp-projects.md)).
- **Copying answers between clean-room siblings.** It is a
  provenance and trust problem, and small communities notice.
- **Trusting a permuter or LLM score.** Reread the C.
- **Replacing a mild fake match with a worse one** to "clean it
  up".
- **Skipping the full image compare** because the function diff is
  empty.
- **Committing a ROM, an extracted asset, or a generated
  `objdiff.json`.**
- **Letting bare numbers stand in for list ids**, which a later
  linking test cannot fix automatically.

## See also

- [04](04-decomp-tooling.md) — tools in the draft → compare → nudge
  loop
- [05](05-ai-llm-decompilation.md) — where AI helps, and the oracle
  lesson
- [07](07-workflow.md) — worked example from PR #34
- [08](08-legal-ethics.md) — safe repo shape
- [16](16-golden-sun.md), [17](17-zelda.md), [25](25-golden-sun-matching-lessons.md),
  [26](26-what-decomp-teaches-about-programming.md)
