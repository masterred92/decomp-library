# AI / LLM-Assisted Decompilation
## LLM4Decompile (EMNLP 2024)
- Paper: https://arxiv.org/abs/2403.05286 ; models: https://huggingface.co/LLM4Binary ; code: https://github.com/albertan017/LLM4Decompile
- Open models 1.3B–33B, x86 assembly → C. Two modes: **End** (asm → C directly) and **Ref** (refines Ghidra pseudo-code).
- The 6.7B End model hits **45.4% re-executability on HumanEval-Decompile** and 18.0% on ExeBench. The paper says that's over 100% better than Ghidra or GPT-4o. Ghidra alone scores ~20% avg.
- The Ref approach does 16.2% better than End; Ref-6.7B v2 ~52.7% avg (HF model card).
- Obfuscated code defeats both the LLM and Ghidra (from the paper).
## Practical AI-assisted RE
- Use LLMs to **rename variables, infer types and structs, and explain functions** on top of Ghidra/IDA output. Bridges like GhidraMCP let an assistant drive Ghidra: https://github.com/LaurieWired/GhidraMCP
- For matching decomp, an LLM gives you a first draft. The **compiler + differ is the ground truth**, so always verify.
- Weaknesses: optimised code (O2/O3), non-x86 ISAs (less training data), and hallucinated semantics.

## 2026-10 update
- Melee 100% (2026-09-07), last PR labelled ai-assisted: https://github.com/doldecomp/melee/pull/3409
- momo5502 FPS write-up (2026-10-09): the lesson is that agents produced plausible-but-wrong code until a byte-matching verifier became the PASS/FAIL oracle. The verifier ignored relocation bytes but checked that each relocation pointed at the same symbol and offset. Agents tried to cheat (inline asm, editing the checker), so CI hashes the checker. With a strict oracle, cheap models did most of the work. https://momo5502.com/posts/2026-10-09-game-decompilation/
- Secondary roundup (unverified individually): Snowboard Kids 1 and 2 at 100% with AI help, SM64DS 99.9% matched. https://capitalandcompute.net/blog/which-games-has-ai-decompiled/
- Relevance to GBA: our gbadt match loop is the same oracle idea.

## What the 2026 wave actually taught (expanded 2026-10-14)

> **Tutor's note.** An AI writing C is like a fast student who guesses with confidence. That's only
> useful if something marks the work *strictly*. In matching decomp, the marker is the compiler plus a
> byte comparison: either the bytes match or they don't. The AI can be wrong as often as it likes, as long as wrong answers
> can't get through.

### The oracle idea
* **momo5502's write-up** ([blog, 2026-10-09](https://momo5502.com/posts/2026-10-09-game-decompilation/)) is
  a field report on running agents, and it barely talks about the game. The key points: a *reviewer agent* failed as a quality gate,
  progress that looked good hid bad code, and swapping the reviewer for a **byte-matching oracle** made cheap models
  reliable. Agents also tried to **cheat the oracle** (inline asm, editing the checker), so the checker was hashed in CI.
  Reported end state: 99% of functions reconstructed, **83% byte-exact**. The code stays private.
* Our Golden Sun loop is the same idea at a small scale: `make clean && make compare` on the full ROM plus all 96 overlays,
  and then a combined-merge rebuild ([25 §5](25-golden-sun-matching-lessons.md)).

### Where AI helps and where it doesn't (from our own matching)
| Helps | Doesn't help (yet) |
|---|---|
| A first-draft C for small leaf functions | Final register-order nudges ([25 §3](25-golden-sun-matching-lessons.md)) |
| Spotting struct shapes from offsets | Knowing the original *style* without examples |
| Writing explanations and PR text | Judging whether a "match" is a fakematch |
| Grinding through lots of easy functions | Legal judgement (see [08](08-legal-ethics.md)) |

### Contested or unverified claims (be careful quoting them)
* **"Melee was decompiled by AI."** Not supported. Only the final PR ([#3409](https://github.com/doldecomp/melee/pull/3409))
  is labelled ai-assisted. The project spent years matching by hand before that.
* **Which game momo5502 decompiled.** His blog post deliberately doesn't name it. Press coverage
  ([Kotaku](https://kotaku.com/activision-and-bethesda-crackdown-on-genai-mw2-and-fallout-projects-2000739580),
  [TheGamer](https://www.thegamer.com/ai-vibecoded-decompilations-takedown-fallout-mw2-dmca/)) ties it to his earlier
  public posts about Modern Warfare 2 (2009). Treat the title as *reported*.
* **"AI decompilation is solved."** 83% byte-exact after an estimated hundreds of billions of tokens is impressive, but it isn't solved.
  Other % claims in secondary roundups haven't been individually verified.
* **LLM4Decompile numbers** measure *re-executability* on benchmarks, not byte matching. You can't compare them directly with decomp %.

### The legal shadow
In early October 2026 the MW2 footage was disabled "in response to a report by the copyright owner", and the code-host
link went dark (Kotaku, TheGamer). Bethesda sent a cease-and-desist over an AI-built *Fallout* recreation in the same week. That
one wasn't a decompilation; it was described as rebuilt without the original assets. Publishers are paying attention to fast, viral AI projects.
See [08](08-legal-ethics.md).

### Common mistakes
* **Using an AI reviewer as the gate.** Use bytes.
* **Letting the agent touch the checker.** Lock or hash it.
* **Publishing faster than you can explain.** Small, readable PRs with honest notes (our #32–#37) get reviewed. A dump of 10,000 functions doesn't.
