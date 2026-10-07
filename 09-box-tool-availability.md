# Tools Installed on This Box (2026-10-07)
Run `source ~/tools/env.sh` first. It sets JAVA_HOME/GHIDRA_HOME/DOTNET_ROOT, puts the tools on PATH, activates the venv, and adds aliases `m2c`, `asmdiff`, `permuter`. Everything in ~/tools takes ~2.2 GB.

| Tool | Version | Path / source | Verified |
|---|---|---|---|
| OpenJDK | 21.0.12 (apt `openjdk-21-jdk-headless`) | /usr/lib/jvm/java-21-openjdk-amd64 | `java -version` |
| Ghidra | 12.1.4 (official NSA GitHub release; SHA-256 ddac49f9…d4db matched) | ~/tools/ghidra_12.1.4_PUBLIC | headless analysis + decompiled `add_mul` |
| radare2 | 6.2.4 (official GitHub .deb, sha256 verified; not in Debian trixie apt) | /usr/bin/r2 | `r2 -v` |
| clang | 19.1.7 | apt | `clang --version` |
| binutils-multiarch, gcc, objdump | apt | /usr/bin | yes |
| MIPS cross gcc/binutils | apt `gcc-mips-linux-gnu` | mips-linux-gnu-* | compiled test.c → MIPS ELF |
| PowerPC cross gcc/binutils | apt `gcc-powerpc-linux-gnu` | powerpc-linux-gnu-* | compiled + objdump |
| jadx | 1.5.6 (sha256 verified) | ~/tools/jadx/bin/jadx | decompiled test .class |
| .NET SDK 8.0.425 + 10.0.401 | dotnet-install.sh | ~/tools/dotnet | yes |
| ilspycmd | 11.1.0 (dotnet global tool) | ~/.dotnet/tools/ilspycmd | decompiled test DLL to C# |
| Python venv | ~/tools/decomp-venv | | |
| ↳ splat64[mips] 0.50.0, spimdisasm, capstone 5.0.9, lief 1.0.0, pyelftools 0.33, pycparser, toml, Levenshtein, colorama, watchdog, cxxfilt | pip | | `splat --help` |
| m2c | git (matt-kempster/m2c) | ~/tools/m2c/m2c.py | decompiled MIPS asm to C |
| asm-differ | git | ~/tools/asm-differ/diff.py | `--help` |
| decomp-permuter | git | ~/tools/decomp-permuter/permuter.py | `--help` |

**Gotcha:** the PyPI package named `m2c` is an unrelated project. The real decompiler is the GitHub repo (or `pip install git+https://github.com/matt-kempster/m2c`).

## Usage
```bash
source ~/tools/env.sh
# Ghidra headless: import, auto-analyse, run a script that prints decompiled C
mkdir -p /tmp/gproj
analyzeHeadless /tmp/gproj P -import ./binary -scriptPath ~/tools/test -postScript Decomp.java -deleteProject
#   (example script ~/tools/test/Decomp.java decompiles function "add_mul"; edit the name)
r2 -qc 'aaa; afl; pdf @ main' ./binary          # radare2 analysis/disasm
mips-linux-gnu-objdump -drz file.o               # MIPS disasm
m2c --target mips-gcc-c func.s                  # asm (glabel format) -> C draft
splat create_config rom.z64 ; splat split rom.yaml
asmdiff -o func_name                            # needs project diff_settings.py
ilspycmd Assembly-CSharp.dll -p -o out/          # Unity Mono -> C# project
jadx -d out/ game.apk                            # Android -> Java
```
Test artifacts are in ~/tools/test (t.c plus x86/MIPS/PPC builds, Ghidra log).
Not installed: IDA, Binary Ninja (commercial), Il2CppDumper/Cpp2IL (Windows/.NET tools; Cpp2IL could be added via dotnet), MWCC/IDO original compilers (projects fetch these themselves).
