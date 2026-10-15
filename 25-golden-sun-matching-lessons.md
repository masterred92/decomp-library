# Golden Sun: lessons from matching functions

Written 2026-10-14 after our first two matching sessions on
[Coaltergeist/goldensun-decomp](https://github.com/Coaltergeist/goldensun-decomp), working from the
[masterred92 fork](https://github.com/masterred92/goldensun-decomp). Background on the project is in
[16-golden-sun.md](16-golden-sun.md), and the general loop is in [07-workflow.md](07-workflow.md).

> **Tutor's note.** "Matching" means writing C that the *original* compiler turns back into the
> *exact* bytes of the cartridge. You don't get partial credit: one swapped register counts as a
> miss. So most of the lessons below aren't about *what* the code does (that part is usually easy).
> They're about *how the original programmers phrased it*, because phrasing is what the compiler
> responds to.
>
> Everything here is described at the C level only. There are no ROM bytes, no dumps of the original
> assembly and no game assets. The C shown is our own reconstruction, taken from the public PRs.

## The scorecard

| PR | Functions | Area |
|---|---|---|
| [#32](https://github.com/Coaltergeist/goldensun-decomp/pull/32) | `HasMove` | units / psynergy |
| [#33](https://github.com/Coaltergeist/goldensun-decomp/pull/33) | `ModifyHP`, `ModifyPP` | unit stats (+ Makefile padding fix) |
| [#34](https://github.com/Coaltergeist/goldensun-decomp/pull/34) | `Func_8012038`, `Func_8012078`, `GetActorSpriteID` | map tiles, actor sprites |
| [#35](https://github.com/Coaltergeist/goldensun-decomp/pull/35) | `Func_80a17c4`, `Func_80a3d6c`, `Func_80b0694` | menu cursor, items, shop |
| [#36](https://github.com/Coaltergeist/goldensun-decomp/pull/36) | 7 battle functions (fades, flash, midpoints, task waits) | battle |
| [#37](https://github.com/Coaltergeist/goldensun-decomp/pull/37) | `Func_8078550`, `Func_807882c`, `Func_8079c8c` | inventory / equipment |
| [#38](https://github.com/Coaltergeist/goldensun-decomp/pull/38)–[#43](https://github.com/Coaltergeist/goldensun-decomp/pull/43) | 19 functions: the former fork branches (djinn, field, party menu, shop, sprite, ui) | various |
| [#44](https://github.com/Coaltergeist/goldensun-decomp/pull/44)–[#49](https://github.com/Coaltergeist/goldensun-decomp/pull/49) | 18 functions | various |
| [#50](https://github.com/Coaltergeist/goldensun-decomp/pull/50)–[#52](https://github.com/Coaltergeist/goldensun-decomp/pull/52) | 19 functions (ui structs, maps, battle); found by clustering sibling functions | various |
| [#53](https://github.com/Coaltergeist/goldensun-decomp/pull/53), [#54](https://github.com/Coaltergeist/goldensun-decomp/pull/54) | 28 functions (22 map overlays, 6 ui/text) | maps, ui |

That's **103 functions across PRs #32–#54**, all still open and awaiting review as of 2026-10-15. The full list is in the fork's
[MATCHED_FUNCTIONS.md](https://github.com/masterred92/goldensun-decomp/blob/main/MATCHED_FUNCTIONS.md). Every PR branch passes the
full compare, and a combined merge of all of them (merge-test6) still matches the ROM and all 96 overlays. Upstream had 1,768
single-function `INCLUDE_ASM` files left, which drops to about **1,665** if everything lands. (Earlier counts in this chapter: 1,745 was
wrong for the first six PRs, and the right figure was 1,749.)

The general programming lessons from all this are in [26](26-what-decomp-teaches-about-programming.md).

---

## 1. The toolchain: know your compiler's quirks

**Compiler:** Camelot's build uses gcc 2.96 (the "camelot" build) in Thumb mode, with the
flag **`-fcall-used-r4`**.

*What that flag means, plainly.* Normally on ARM, registers r4–r7 are "callee-saved": if a
function uses them, it has to put back what was there before. `-fcall-used-r4` tells the compiler that **r4
is scratch** too, so any function call may wipe it. As a result:

* A value that must **survive a function call** lands in **r5** (or higher), not r4.
* So when your attempt keeps a value in r4 across a call and the original uses r5, the fix is
  usually not to fight the register. Instead, check whether that value really lives across a call in
  the original logic.

**The section-padding fix ([#33](https://github.com/Coaltergeist/goldensun-decomp/pull/33)).**
`ModifyPP` is the last function in its section (`.text.party_2`). When the assembler (gas) padded
the end of the section out to alignment, it filled the gap with a **Thumb NOP**, but the cartridge
has **zeros** there. The C was correct, and the mismatch was all in the padding. The fix was
to make the Makefile's appended alignment also zero-align *whichever section is current at end of
file*. It's byte-neutral everywhere else.

> Lesson: when everything matches except a couple of bytes at the very end of a section,
> suspect alignment and padding, not your C.

---

## 2. Patterns: how Camelot's C was phrased

These go roughly from "easy to spot" to "subtle."

### 2.1 Struct fields beat raw offsets and masks
The data only matched once it was described as **structs** (and, where needed, bitfields). Writing
out the raw maths with the same meaning didn't match.

* `HasMove` ([#32](https://github.com/Coaltergeist/goldensun-decomp/pull/32)): the psynergy slots are
  4 bytes apart, but every access is a halfword. Modelling them as
  `struct UnitMove { u16 id; u16 __unk2; }` makes the loads come out right.
* Map layers, sprite flags and unit moves all behaved the same way.

**Why:** the compiler picks load sizes, offsets and sometimes the order of address maths from the
*types*. With the right type, it emits the right instructions without extra help.

### 2.2 Signedness is visible: `s16` → `ldrsh`
In `ModifyHP`/`ModifyPP` ([#33](https://github.com/Coaltergeist/goldensun-decomp/pull/33)), HP, PP,
maxHP and maxPP are read with a **signed** halfword load (`ldrsh`). An unsigned `u16` gives
`ldrh`. So those `struct Unit` fields became `s16`. The load instruction tells you what type a field has.

### 2.3 The order you write things in is the order you get
gcc 2.96 doesn't reorder much, so two expressions that mean the same thing can compile differently:

* `total - f(x) + 15` matched; `total + 15 - f(x)` didn't.
* Map tiles ([#34](https://github.com/Coaltergeist/goldensun-decomp/pull/34)) only matched in two steps:
  ```c
  tiles += x + (z << 7);
  return tiles->unk2;
  ```
  The one-liner `tiles[x + (z << 7)].unk2` gives the same answer with different instructions.

### 2.4 Move a pointer forward vs index an array
This is related to 2.3. Advancing a pointer (`p++` / `p += n`) and indexing (`p[i]`) put the pointer
increment in different places in the loop or expression. When the original bumps a pointer, you
need to bump one too.

**This depends on context.** Rule 2.10 below points the *other* way. In `Func_80c1084` (#36), the code steps through *fields of
one struct* (`p += 0x64e / 2;` then reads `*p`), and moving the pointer forward matched. In the party-menu loops over a
**table of pointers**, `table[i]` matched and pointer-walking didn't. So the question to ask is whether you're
moving through one block of data or looking up entries in a table.

### 2.5 Load into named locals first
Camelot's programmers used lots of small, named temporaries. Copying values into locals before using them
(the four coordinates in a midpoint calculation, an actor's first sprite layer, `x >>= 20; z >>= 20;`
at the top of the tile functions) kept fixing the register order.

### 2.6 Extra variable steps change register allocation (`GetActorSpriteID`)
`GetActorSpriteID` ([#34](https://github.com/Coaltergeist/goldensun-decomp/pull/34)) only matched with
what looks like "unnecessary" steps: a separate `base`, a separate `ofs`, then the actor pointer, then a
`layer` local, then `spriteID = (s16)layer->spriteID`. Each extra variable changes how long a value
lives, and that changes which register it gets. Treat local variables as a tool for controlling
register allocation, not just for readability.

### 2.7 An `int` temp avoids an extra `mov` when masking
When masking a value (e.g. `id & 0x3fff`), putting the result in an `int` temporary instead of a
narrower type kept the compiler from adding an extra `mov` or sign/zero-extension.

### 2.8 Clamp-check order sets branch order
`ModifyHP`/`ModifyPP` clamp to `[0, max]` through a small `static inline Clamp`. Checking
`< 0` first vs `> max` first changes the order of the branches in the output, so check them in the
order the original does.

### 2.9 The Thumb return-type tell: `pop {r0}` vs `pop {r1}`
In Thumb, a function that saved `lr` returns by popping it into a free low register and branching.
gcc 2.96 picks that register based on whether **r0 holds a return value**:

* `pop {r0}` → r0 is free → the function is `void`.
* `pop {r1}` → r0 is "taken" → the function returns something.

`Func_80be02c` and `Func_80b6378` ([#36](https://github.com/Coaltergeist/goldensun-decomp/pull/36))
end with `pop {r1}` but never compute a return value. They matched only when declared `int` with
**no `return` statement**, which looks like old **implicit-int** C (functions declared without a type
default to `int`). It's an old habit, and the last instruction gives it away.

### 2.10 Index style for loops over pointer tables
Four party-menu functions (fork branch `party-menu-batch2`) loop over a table of cursor pointers. They only matched in index style:
```c
for (i = 0; i <= 31; i++) {
    struct MenuCursor *cursor = cursors[i];
    if (cursor != 0) { ... }
}
```
Walking a pointer (`*cursors++`) moves the increment and changes the registers. See the note under 2.4 for when each style wins.

### 2.11 Don't save what the original re-reads
`GiveDjinni` was a near-miss (one extra `mov`) until we **stopped** saving `u->numDjinn[element]` in a local and read the
field again each time it's used:
```c
if (u->numDjinn[element] > 9)
    return -1;
...
u->numDjinn[element]++;
```
The permuter found this one. It goes against 2.5, but it fits the same principle: write what the original
*did*. If it loaded the field twice, you load it twice.

### 2.12 Keep a tested-and-passed value in a named local
In `Func_80b26cc` (fork branch `shop-batch1`), the item is checked and then passed on, so it needs its own variable,
assigned inside the test:
```c
for (i = 0; i < 8 && (item = gShopItemLists[shop][24 + i]) != 0; i++)
    _Func_8078ad0(item, 1);
```
Reading the table a second time for the call doesn't match.

### 2.13 Loop shape: counted `for` with an early `break`; length check first
* `Func_8012350`: a wait-loop with a maximum of 300 tries matched as a counted `for` with an early `break`. The `while` version
  that means the same thing didn't:
  ```c
  for (i = 0; i < 300; i++) {
      if (state[1] <= 0xff && state[2] <= 0xff)
          break;
      WaitFrames(1);
  }
  ```
* `Func_80b2720`: in a combined loop condition, put the **length check first**:
  `for (n = 0; n < 24 && gShopItemLists[shop][n] != 0; n++)`. Swapping the order of the `&&` swaps the branch order (like 2.8).

### 2.14 Patterns from the 103-function run (ranked by how many matches each unlocked)
* **`z - 1` as an expression, not `z--`.** Writing `f(x, z - 1, ..., z - 1)` instead of decrementing `z` first
  fixed an r5/r6 swap across a whole family of map overlays (about 9 functions, [#53](https://github.com/Coaltergeist/goldensun-decomp/pull/53)).
  Decrementing changes the variable, so the compiler has to keep the new value somewhere. An expression doesn't change anything.
* **A `const` alias for the overlay division helper.** Map overlays divide through `_divsi3_RAM`, a copy of the
  division routine in RAM. Declaring it as a pure function let the compiler schedule it the way the original did (6 functions):
  ```c
  extern int _divsi3_RAM__c(int, int) __asm__("_divsi3_RAM") __attribute__((const));
  a[0x44 / 4] = vx - _divsi3_RAM__c(vx, 18);
  ```
* **Recompute from the base.** With struct views that contain array or struct members, write every access
  as `base->member[i]` from the same base, not as chained offsets from a previous pointer (about 5).
* **An unprototyped `extern void f();`** when the same file defines `f` later with parameters. Old C allowed calls without
  a full prototype, and that changes how arguments are set up (1, `Func_809a65c`).
* Smaller wins: a 2-bit bitfield instead of a hand-written mask; `x >>= 3; y >>= 3;` as separate statements before a call;
  `param -= K;` in place; masking into a named local before OR-ing.

---

## 3. What didn't match, and why

Almost every miss had **the right instructions in the wrong registers or order**:

* `Func_8015e8c` (ui/window, free-list pop): r1/r2 swapped. A 5-minute
  [decomp-permuter](https://github.com/simonlindholm/decomp-permuter) run found a "best" result that
  was **semantically wrong**, so we threw it out. A permuter score isn't proof that the code is correct.
* `Func_8092b54` (copies a sprite slot + 10-bit OAM tile): load scheduling differs.
* `Func_801a7c0`, `Func_80a7440`, `Func_8079bf8`: close, with small differences in how arguments are copied or instructions scheduled.
* `GiveDjinni`: one extra `mov r3, r4` before a compare.
* **Skipped on purpose:** `Func_80c0eb8` (would need an inline `stmia` trick like an existing
  fakematch), `GetSpriteVoiceEntry` (its `.s` also carries read-only data, so it needs splitting first),
  `Func_80bd850` (r9 saved around `_call_via_r3`).

**Policy: no new fakematches.** A fakematch is C that produces the right bytes through tricks
(dummy variables, `asm` statements, nonsense casts) instead of plausible original code. The
project tracks the ones it already has. We didn't add any: if it didn't match honestly, it stayed in assembly.

---

## 4. Data-layout findings (C-level struct shapes)

These are partial views, kept local to their files unless noted, and named only as far as the evidence goes.

* **Map layers** ([#34](https://github.com/Coaltergeist/goldensun-decomp/pull/34)): tiles are 4 bytes
  (`MapTile`, byte 2 is read/written by the getter/setter); the tile map is **128 wide**
  (`x + (z << 7)`) and coordinates are fixed-point with the tile index in the top bits (`>> 20`). Four
  layers of **0x30 bytes** each sit at **+0x130** of the field state (`layer & 3`); the getter falls back to
  `gBuffer` when that state is null.
* **Actors** ([#34](https://github.com/Coaltergeist/goldensun-decomp/pull/34)): an actor table is
  indexed by `(actorID & 0xfff) * 4 + 0x14`; an actor has a `visible` flag and a sprite whose
  `layers[0]` holds a signed `spriteID` (-1 = none).
* **Units** ([#32](https://github.com/Coaltergeist/goldensun-decomp/pull/32),
  [#33](https://github.com/Coaltergeist/goldensun-decomp/pull/33),
  [#35](https://github.com/Coaltergeist/goldensun-decomp/pull/35),
  [#37](https://github.com/Coaltergeist/goldensun-decomp/pull/37)): 32 psynergy slots of
  `{u16 id; u16 unk;}` with the move ID in the low 14 bits; HP/PP/max values are `s16`; the first 15
  item slots hold item IDs in the low 9 bits (`& 0x1ff`), with bit `0x200` = equipped; a class byte at
  0x129.
* **Menu cursor** ([#35](https://github.com/Coaltergeist/goldensun-decomp/pull/35)): x/y are copied into
  an OAM "shadow" at +0x10 (attr0 y is 8 bits, attr1 x is 9 bits), and the affine-mode/matrix bits are cleared.
* **Shop** ([#35](https://github.com/Coaltergeist/goldensun-decomp/pull/35)): a list starting at
  +0x348, with the current index as a halfword at +0x39e.
* **Battle** ([#36](https://github.com/Coaltergeist/goldensun-decomp/pull/36)): fades step BLDALPHA 2 per
  frame; a flash indexes a 16-entry signed BLDY table by a halfword at +0x64e of the battle state; a
  state value at +0x800 is waited on.

---

## 5. Workflow that worked

1. **Triage a worklist.** List every unmatched single-function `INCLUDE_ASM` file (1,768). Drop
   known unmatchable/fakematch entries, anything touched upstream in the last 14 days (to avoid
   colliding with someone else's work), our own open PRs and trivial veneers, which leaves **1,413**. Rank them by
   `instructions + 3×branches + 2×calls` and start at the easy end. Also note which ones already have a
   parked `non_matching` candidate, because those are half-done.
2. **Small, themed batches.** 1–7 functions per PR, grouped by file or area, with no overlapping header
   changes between PRs so each one can be merged on its own. We keep the number of open PRs small so we don't swamp
   the maintainers.
3. **Explain any header change in the PR**, and offer an alternative (e.g. the raw-pointer form in #32).
4. **No generated files.** Upstream removed its progress script, so PRs contain only source changes and
   maintainers update the progress numbers themselves.
5. **Full `make clean && make compare`** on every branch: the main ROM *and* all 96 overlays must be OK.
6. **Re-verify from scratch, then merge everything together.** Before reporting, every PR branch was rebuilt
   cleanly, and then all six were merged into one throwaway branch and rebuilt. All seven builds
   (6 branches + the combined merge) produced an identical ROM that matched the original. That proves the PRs don't
   conflict with each other.

---

## 6. Camelot's coding style, in one paragraph

Camelot wrote **plain, step-by-step C**. They loaded values into small named locals, walked pointers
forward and described data with structs instead of raw offsets. Their types were exact (signed where the game
needs negatives), they wrote expressions in the order they thought of them, and sometimes they used old
implicit-int habits (functions that "return int" but don't). gcc 2.96 doesn't rearrange much, so the
code's phrasing shows through in the output. That's good news for the remaining ~1,740 functions: once you
know the habits, most functions should come out the same way.
