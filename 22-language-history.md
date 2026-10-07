# 22 — A History of Languages: From Machine Code to Lingua Francas

Tutor note: this one's a story, not a lab. It explains *why* the code we decompile looks the way it does.

## 1. Machine code (1940s)
The first computers (ENIAC, the Manchester Baby, EDSAC) were programmed in raw numbers, either by setting switches or by writing binary opcodes. Every CPU has its own set. That's still what a ROM holds today, and it's what we read in Ghidra.

## 2. Assembly (late 1940s–1950s)
Short names (`ADD`, `LDR`, `JMP`) replaced the numbers, and an *assembler* translated them back. Kathleen Booth is credited with an early assembly language (1947), and EDSAC had "initial orders" (1949). Assembly is still one name per instruction. Game Boy games were mostly written this way, which is why GBC decomps are disassemblies (note 19).

## 3. The first high-level languages (1950s)
- **FORTRAN** (1957, John Backus at IBM): formulas written like maths. Its compiler optimised so well that people trusted compilers for the first time.
- **Lisp** (1958, John McCarthy): code as lists, recursion, garbage collection. It's the ancestor of functional programming and was the language of early AI.
- **COBOL** (1959, CODASYL, building on Grace Hopper's FLOW-MATIC): English-like business code. Huge amounts of it still run banks today.
- **ALGOL 60**: block structure and formal grammar (BNF). Most later languages descend from it.

## 4. C and Unix (1969–1978)
Dennis Ritchie made C at Bell Labs (1972) so Unix could be rewritten from assembly in something portable. It came via BCPL and Ken Thompson's B. Kernighan & Ritchie's book (1978) spread it, and ANSI standardised it in 1989.

### Why C became the lingua franca of systems and games
- **Close to the metal:** each line maps predictably to a few instructions. That's why *matching decomp works*, since you can rebuild the exact bytes.
- **Portable:** a C compiler (later GCC, 1987) existed for nearly every CPU, including MIPS, PowerPC and ARM.
- **Small runtime:** it fits on a 32 KB GBA work-RAM budget.
- **Network effect:** console SDKs (Nintendo, Sony) shipped C libraries, so studios wrote C, then C++.
- **The ABI is the common tongue:** even today Python, Rust and C# talk to each other through C calling conventions.

## 5. Later languages (short tour)
Smalltalk (1972, objects) → C++ (1985, Stroustrup: C plus classes, used by most 2000s console games) → Python (1991, van Rossum) → Java (1995, run anywhere on a VM) → JavaScript (1995, Eich, written in 10 days) → C# (2000, Unity's language) → Go (2009, Google) → Rust (2015 1.0, memory safety without garbage collection) → TypeScript (2012, JS with types).

## 6. Human lingua francas
A *lingua franca* is a shared language between groups with different native tongues. The term itself comes from a Mediterranean trade pidgin (Romance plus Arabic, Greek and Turkish words) used from roughly the 11th to the 19th century.
- **Greek (Koine):** after Alexander, across the eastern Mediterranean. It's the language of the New Testament.
- **Latin:** the Roman Empire, then the medieval Church and scholarship until about the 1700s (Newton's *Principia* was in Latin).
- **Arabic:** the Islamic world from the 7th century on, covering science, maths ("algorithm" comes from al-Khwarizmi) and trade.
- **French:** diplomacy from the 17th to the early 20th century.
- **English:** British Empire, then US economic and technical power. It's the language of the internet and of programming keywords.
- Others: Swahili, Persian, Malay, Mandarin, Sanskrit.

### Pidgins and creoles
A **pidgin** is a simplified contact language with no native speakers (e.g. early Tok Pisin). When children grow up speaking it, it becomes a **creole** with full grammar (Haitian Creole, Jamaican Patois, modern Tok Pisin). Coding parallel: pseudo-code and assembly macros are "pidgins", and languages like C became full "creoles" passed down to new programmers.

## 7. LLMs as translators
LLMs are trained on human text *and* code. Their tokenizers (see karpathy/minbpe) split both into the same kind of pieces, so English, Python and assembly sit in one shared space. That's why a model can turn assembly into C (LLM4Decompile, note 05) or English into code. In effect, English is becoming a programming language, and the LLM is the new compiler front end. The difference from a real compiler is that its output isn't guaranteed correct, which is why our *compile and compare* loop matters.

## Sources
- Computer History Museum, "Timeline of Computer History": https://www.computerhistory.org/timeline/
- D. M. Ritchie, "The Development of the C Language" (1993): https://www.bell-labs.com/usr/dmr/www/chist.html
- J. Backus, "The History of FORTRAN I, II and III", ACM HOPL (1978)
- J. McCarthy, "History of Lisp" (1979): http://jmc.stanford.edu/articles/lisp/lisp.pdf
- ACM History of Programming Languages conference proceedings (HOPL I–IV)
- N. Ostler, *Empires of the Word* (2005) and *The Last Lingua Franca* (2010)
- Britannica, "Lingua franca", "Pidgin", "Creole languages": https://www.britannica.com/topic/lingua-franca
- J. Holm, *An Introduction to Pidgins and Creoles* (Cambridge UP, 2000)
- Vaswani et al., "Attention Is All You Need" (2017): https://arxiv.org/abs/1706.03762
- Tan et al., "LLM4Decompile" (2024): https://arxiv.org/abs/2403.05286
