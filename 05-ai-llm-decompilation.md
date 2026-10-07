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
