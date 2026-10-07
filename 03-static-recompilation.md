# Static Recompilation & Source Ports
Static recomp **translates machine code to C/C++ automatically**, without hand-written source. It's faster to get a native port than with matching decomp, but the output isn't readable source.

- **N64Recomp**: "statically recompile N64 binaries into C code that can be compiled for any platform." Inspired by the IDO static recompilation used for matching decomp; cites jamulator (NES) as prior art. Powers Zelda 64: Recompiled. https://github.com/N64Recomp/N64Recomp , https://github.com/Zelda64Recomp/Zelda64Recomp
- **XenonRecomp**: converts Xbox 360 (PowerPC) executables to C++; x86-only output for now (uses x86 intrinsics); needs Clang 18+. Used by Unleashed Recompiled (Sonic Unleashed). https://github.com/hedge-dev/XenonRecomp , https://github.com/hedge-dev/UnleashedRecomp
- **Ship of Harkinian** (HarbourMasters): a PC port of OoT built **on the matching decomp** (not recomp), and you need your own ROM. Same group did 2 Ship 2 Harkinian (MM). https://github.com/HarbourMasters/Shipwright
- Related: PS2Recomp efforts, and the SM64 PC port from the decomp.

Key point: decomp → readable source → real ports/mods; recomp → fast native ports, plus mod hooks.
