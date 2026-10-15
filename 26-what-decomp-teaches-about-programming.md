# 26: What matching decomp teaches about programming

Written 2026-10-15 after matching **103 Golden Sun functions** (PRs
[#32](https://github.com/Coaltergeist/goldensun-decomp/pull/32)–[#54](https://github.com/Coaltergeist/goldensun-decomp/pull/54);
full list in the fork's [MATCHED_FUNCTIONS.md](https://github.com/masterred92/goldensun-decomp/blob/main/MATCHED_FUNCTIONS.md)).
[Chapter 25](25-golden-sun-matching-lessons.md) lists the specific matching tricks. This chapter is about the general programming
lessons behind them, the kind that apply to any code you write.

> **Tutor's note.** Matching decomp is a strange way to learn programming. You aren't trying to make code
> that works, because the game already works. You're trying to work out *exactly* what someone wrote 25 years ago,
> and the only feedback is whether the compiler produces the same bytes. That strictness is what makes it so
> instructive: every gap between "means the same" and "is the same" becomes visible.
>
> All C below is ours, from the public PRs. There are no ROM bytes and no original assembly.

---

## 1. Source code is instructions to a compiler, not the program

The cartridge doesn't contain C. It contains machine code, and the C is a set of instructions *for producing* that machine code.
Several things happen in between, and matching makes you see each of them:

* **Registers.** The CPU works on a handful of fast slots (r0–r7 for most Thumb instructions). Your variables don't
  "exist" in memory; the compiler decides which register holds what, and when. Most of our near-misses were just two values
  in each other's registers ([25 §3](25-golden-sun-matching-lessons.md)).
* **The calling convention.** It's an agreement about how functions hand data to each other: arguments in r0–r3, the result in r0,
  and some registers that must survive a call. Golden Sun's compiler flag `-fcall-used-r4` changes the agreement, so anything that
  has to live across a call moves to r5 or above ([06](06-platform-notes.md)).
* **The stack.** When there aren't enough registers, or a function needs more than four arguments, values go to memory
  on the stack. Saving and restoring registers there is the "push … pop" at the start and end of every function.
* **The literal pool.** Thumb instructions can only hold small numbers, so big constants and addresses are stored
  next to the function and loaded from there. Whether a constant is pooled or built with `mov` depends on *how you wrote the store*:
  through a struct field or through a cast pointer ([PLAYBOOK](https://github.com/masterred92/gba-decomp-toolkit/blob/main/accel/PLAYBOOK.md)).

**Lesson for any programmer:** a high-level line has a cost and a shape underneath it. You don't need to think about
that most of the time, but knowing it's there is the difference between guessing at performance and understanding it.

## 2. Same meaning, different code

Two pieces of C can mean exactly the same thing and still compile differently. A few examples:

```c
/* Map tile lookup (#34): only the two-step version matched */
tiles += x + (z << 7);
return tiles->unk2;
/* ...not: return tiles[x + (z << 7)].unk2; */

/* Map overlays (#53): an expression, not a decrement, fixed a register swap */
__Func_8010704(x, z - 1, 1, 3, ..., z - 1);
/* ...not: z--; __Func_8010704(x, z, ...); */
```

Loop shape matters too: a counted `for` with an early `break` compiles differently from a `while` that does the same thing
(`Func_8012350`, [25 §2.13](25-golden-sun-matching-lessons.md)).

**What this means for normal programming:**
* **Write for readers first.** Modern compilers (with optimisation on) usually turn equivalent phrasings into the same code,
  so pick the clearest one. gcc 2.96 was simpler, which is why phrasing shows through so much here.
* **But don't assume the compiler is magic.** When performance matters, *measure* and look at the output (tools like
  Compiler Explorer exist for this). Matching taught us to read compiler output without being afraid of it.
* "Equivalent" is a claim you should be able to check. Matching checks it with bytes. Normal code checks it with tests (§7).

## 3. Data structures shape the code

The biggest single unlock in matching was **describing the data correctly**. Once the data had the right shape, the code
around it often just matched:

```c
/* A unit's psynergy slots (#32): 4 bytes apart, but only the first halfword is the move */
struct UnitMove { u16 id; u16 __unk2; };

/* Map layers (#34): partial views, naming only what we understand */
struct MapLayer { struct MapTile *tiles; unsigned char pad4[0x2c]; };
struct MapLayerState { unsigned char pad0[0x130]; struct MapLayer layers[4]; };
```

* **Structs** tell the compiler (and the reader) which fields belong together and what size they are.
* **Arrays** say "many of the same thing, one after another". That's why `s->arr[i]` and pointer arithmetic produce different loops.
* **Bitfields** pack small values into one word; a 2-bit field at bit 2 replaced a hand-written mask ([25 §2.14](25-golden-sun-matching-lessons.md)).
* **Partial views** (`pad` bytes for the parts we don't know yet) let you be honest about what you've figured out.

**Lesson:** "show me your data structures and I'll understand your program" is an old saying for good reason. When code
feels tangled, the fix is often in the data layout, not the logic.

## 4. Types are part of the meaning

* **Signed vs unsigned.** HP is stored as `s16` because the game loads it with a sign-extending instruction ([#33](https://github.com/Coaltergeist/goldensun-decomp/pull/33)).
  Choosing the wrong one changes which comparisons work and how overflow behaves.
* **`int` vs `short`.** Narrow types make the compiler add extra instructions to trim values back to size. An `int` temp avoided an
  extra instruction when masking. In everyday code: use the natural word size for locals and keep narrow types for storage.
* **Return types.** A function's ending shows whether the compiler thought it returns a value ([25 §2.9](25-golden-sun-matching-lessons.md)).
* **Implicit int and missing prototypes.** Older C let you declare functions with no return type (assumed `int`) and call
  functions without saying what parameters they take (`extern void f();`). Both matched Camelot's code in places
  ([#36](https://github.com/Coaltergeist/goldensun-decomp/pull/36), [#53](https://github.com/Coaltergeist/goldensun-decomp/pull/53)).
  Modern C and C++ make these errors or warnings, and that's an improvement: a prototype is a promise the compiler can check.

**Lesson:** types aren't paperwork. They decide what the machine actually does.

## 5. Reading other people's code: "write it the way they did"

The core rule of matching is to write the code *the way the original author did*, not the way you would.
After 103 functions, a picture of Camelot's programmers emerges: plain, step-by-step C, small named locals, pointers
walked forward through structs, index loops over tables, fields re-read rather than cached, and some
old-fashioned habits ([25 §6](25-golden-sun-matching-lessons.md), [20](20-developer-approaches.md)).

That's a kind of **empathy for the author**. You stop asking "why is this written so oddly?" and start asking
"what was this person thinking, with their tools, under their deadlines?" The same skill applies whenever you
join a project, review a PR or fix legacy code: understand the existing style before you change it, and keep new code
consistent with it.

## 6. Debugging as a hypothesis loop

Every near-miss followed the same loop:
1. **Diff:** look at exactly where your output and the target differ.
2. **Theory:** "the registers are swapped because my value lives longer than theirs."
3. **Change one thing:** e.g. turn `z--` into `z - 1`.
4. **Measure:** did the diff count go down, up, or move somewhere else?

Changing several things at once makes the result unreadable. A tiny diff (2 instructions) that won't close is useful information too:
we park it and come back later instead of forcing it. This is ordinary scientific method, and it's the same loop for any bug:
reproduce it, form a theory, change one variable, measure.

## 7. Verification culture

* **Byte-exact checks.** `make clean && make compare` rebuilds the whole ROM and all 96 overlays and compares hashes. "It looks right" counts for nothing.
* **Merge tests.** Each PR passes on its own, but we also merge all open branches together and rebuild (merge-test1 … merge-test6). That's how
  we caught a real conflict in `party_menu.c` before any maintainer had to deal with it.
* **A checker that can't be tampered with.** In momo5502's AI project, agents tried to edit the checker to pass ([05](05-ai-llm-decompilation.md)).
  The fix was to hash the checker in CI. The general rule is that whatever gives the final verdict must sit outside the reach of whatever is being judged.

**For ordinary software**, the same ideas show up as automated tests (your "compare"), CI that runs on every change (your
"merge test") and protected test suites (nobody edits a test just to make it pass). Matching decomp is an extreme version of a
well-tested project: the test suite is the original program.

## 8. Automation and scaling: the 80/20 lesson

Speed over time (from our own logs):
* In early sessions, each function took a few minutes of hand work (roughly 2.7 min each).
* By batch 3 (2026-10-15), matching took ~24 s per function, or ~2.4 min each counting verification, PRs and the index.
* In the latest run, 28 functions took 13 minutes including PRs (~28 s each).

What made the difference, in the [toolkit's accel/](https://github.com/masterred92/gba-decomp-toolkit/tree/main/accel) scripts:
* **Patterns written down** ([PLAYBOOK.md](https://github.com/masterred92/gba-decomp-toolkit/blob/main/accel/PLAYBOOK.md)), so each lesson only has to be learned once.
* **Clustering** (`cluster.py`): finding sibling functions with the same shape as one already matched. One solved function becomes a template for the whole family.
* **Templates and skeletons** (`skel.py`): a first draft with the obvious fixes already applied.
* **Automatic mutation** (`mutate.py`): it tries the playbook's rewrites one at a time and keeps whatever lowers the diff.

**What worked and what didn't:** clustering and templates did most of the work. Automatic mutation *alone* rarely finished
a function. In the latest log, none of 93 automatic attempts matched on their own, while a human (or assistant) applying a
pattern did. The permuter helped occasionally, on register-only differences.

**The 80/20 lesson:** most functions fall to a few common patterns, quickly. The last 20% (scheduling quirks, pooled literals)
takes most of the time, and sometimes the right call is to park it. Automate the repetitive middle, and keep human judgement for the rest.

## 9. Collaboration

* **Small, themed PRs**: 1–22 functions each, grouped by file, with a short explanation of any header or struct change.
* **No generated files**: upstream generates its own progress numbers, so we don't commit them.
* **Check for overlap** with other contributors' open work before starting. Anything touched upstream in the last 14 days is skipped.
* **Branch hygiene**: one `kenny/…` branch per PR, based on upstream main, and stacked only when a PR really depends on another.
* **Mistakes we made and caught:**
  * *Staging too broadly* (`git add src`) can sweep scratch files into a commit. Stage specific files and read `git diff --staged` before committing.
  * *Stray branches*: some batches were pushed without the `kenny/` prefix and duplicated other branches. We confirmed each was identical to a kept branch or already inside an open PR before deleting it, so nothing was lost.
  * *Count errors*: we once published "1,745 remaining" when it should have been 1,749. Now the count comes from the diffs
    (`-INCLUDE_ASM` lines), not mental arithmetic, and the correction is noted openly in [25](25-golden-sun-matching-lessons.md).

**Lesson:** being reliable as a collaborator matters as much as being clever. Reviewers trust contributors who make their work easy to check.

## 10. Taking this beyond decomp

* **WoW addons** ([wow-forever-addons](https://github.com/masterred92/wow-forever-addons)): the same verification culture applies.
  Each API call was checked against Blizzard's published UI source, with a linter and mock tests. The README says up front what
  couldn't be tested in-game, just as a parked near-miss is flagged as unfinished.
* **Your own game design** ([21 Stage 10](21-learning-paths.md)): designing your data first (§3) and choosing types deliberately (§4)
  are where good game code starts. Reading Camelot's code shows how a small team built a big RPG out of simple, consistent pieces.
* **Learning to program generally**: [21](21-learning-paths.md) (paths), [22](22-language-history.md) (why C looks the way it does), [24](24-linux-and-torvalds.md) (Git and collaboration culture).

See also: [04](04-decomp-tooling.md) tools, [06](06-platform-notes.md) GBA/Thumb, [07](07-workflow.md) workflow, [25](25-golden-sun-matching-lessons.md) matching patterns.
