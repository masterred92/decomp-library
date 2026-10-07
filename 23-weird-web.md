# 23: Weird Web, Sites That Break the Rules

This is a study list of open-source websites and web experiments that do things a browser was never meant to do: games in the URL bar, Doom built from CSS, Linux running inside a PDF, one checkbox grid shared by millions of strangers. Every entry here has public source code. The live URL is listed when one exists.

**Why this belongs in a decomp library:** a lot of these projects use the same skills as decompilation. You read a strange system closely, find the one thing it can really do, and build a whole program on top of that. Several of them are literally ports of old C games to the web (see [03-static-recompilation.md](03-static-recompilation.md)), and the emulators are a good next step after [06-platform-notes.md](06-platform-notes.md). The rest teach rendering tricks, tiny code, and game feel.

Checked 2026-10-07: every repo was confirmed to exist with `gh api repos/OWNER/NAME`. Star counts change over time, and some sites go offline, so check live links before relying on them.

## Study these 5 first

| # | Repo | Live | Why first |
|---|---|---|---|
| 1 | [ading2210/doompdf](https://github.com/ading2210/doompdf) | [doompdf.pages.dev/doom.pdf](https://doompdf.pages.dev/doom.pdf) | A C game compiled for a strange target (JavaScript inside a PDF). It's the clearest example of "port a C codebase to a hostile platform", which is what decomp ports do. |
| 2 | [d07RiV/diabloweb](https://github.com/d07RiV/diabloweb) | [d07riv.github.io/diabloweb](https://d07riv.github.io/diabloweb/) | Built on **devilution**, the Diablo decompilation, compiled to WebAssembly. It goes all the way from decompiled source to a game that plays in a browser, and it uses a bring-your-own-data model like ours. |
| 3 | [copy/v86](https://github.com/copy/v86) | [copy.sh/v86](https://copy.sh/v86/) | A full x86 PC emulator that translates machine code to WebAssembly while it runs (a JIT). You can watch CPU emulation and dynamic recompilation happen in a browser tab. |
| 4 | [NielsLeenheer/cssDOOM](https://github.com/NielsLeenheer/cssDOOM) | [cssdoom.wtf](https://cssdoom.wtf) | Reads Doom's level data and builds the 3D world from CSS `div`s and trigonometry. It teaches how classic game data (vertices, lines, sectors) turns into a picture on screen. |
| 5 | [epidemian/snake](https://github.com/epidemian/snake) | [demian.ferrei.ro/snake](https://demian.ferrei.ro/snake) | Snake drawn in the address bar using Braille characters. It's small enough to read in one sitting, and it shows the "find a weird output channel" mindset. |

## 1. Abusing browser features

These projects take a part of the browser you never think of as a screen and draw on it anyway.

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [epidemian/snake](https://github.com/epidemian/snake) | [demian.ferrei.ro/snake](https://demian.ferrei.ro/snake) | Snake played in the URL bar | Each Braille character (⠁⠃⠇…) is a 2×4 dot grid, so a short string can work as a tiny bitmap. The game redraws by rewriting the URL's `#hash` many times a second with `history.replaceState`, which changes the URL without reloading the page. | JS | Treat any text field as a framebuffer. Packing pixels into bits is how GBA tiles work too. |
| [MatthewRayfield/url-bar-games](https://github.com/MatthewRayfield/url-bar-games) | [article](http://matthewrayfield.com/articles/games-and-graphics-in-popup-url-bars/) | Games and animations inside the URL bars of several popup windows | Opens small popups and updates each one's URL as a row of "pixels". Stacking popups gives more rows. | JS | Thinking about the display first: the screen is whatever you can redraw fast enough. |
| [MatthewRayfield/popup-trombone](https://github.com/MatthewRayfield/popup-trombone) | [play](http://matthewrayfield.com/goodies/popup-trombone/) | An instrument you play by resizing a popup window | Reads the window's size and position (`outerWidth`, `screenX`) and maps them to pitch with the Web Audio API. | JS, Web Audio | Unusual inputs. Mapping one number to a sound is the basis of game audio feedback. |
| [MatthewRayfield/inspect-this-snake](https://github.com/MatthewRayfield/inspect-this-snake) | n/a | Snake played inside the browser's DevTools element inspector | The game rewrites DOM nodes and attributes, so the drawing shows up in the Elements panel, not on the page. | HTML, JS | DevTools are views of live data structures, which is how you'll use debuggers and memory viewers in emulators. |
| [nolenroyalty/faviconic](https://github.com/nolenroyalty/faviconic) | [blog](https://eieio.games/blog/running-pong-in-240-browser-tabs/) | Pong running across the favicons of 240 tabs | Each tab's favicon is one "pixel". The tabs keep in sync and redraw their icons. A script opens all the windows (macOS only). The author's own README says the code is rough. | JS, AppleScript | Keeping many independent processes in step, and working within extreme limits. |
| [bgstaal/multipleWindow3dScene](https://github.com/bgstaal/multipleWindow3dScene) | n/a (clone and open `index.html`) | One 3D scene spread across several browser windows, which react when you drag them | Each window writes its screen position to `localStorage`. The others read it and offset their three.js camera, so the windows look like peepholes into one shared world. | JS, three.js | Shared state between processes, plus camera and screen-space math. |
| [tholman/elevator.js](https://github.com/tholman/elevator.js) | [tholman.com/elevator.js](http://tholman.com/elevator.js) | A "back to top" button that plays elevator music while it slowly scrolls up | Eased scrolling timed to an audio clip, then a "ding". | JS | Game feel on a website: timing, easing and sound turn a boring action into a joke. |

## 2. Running where you don't expect

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [ading2210/doompdf](https://github.com/ading2210/doompdf) | [doom.pdf](https://doompdf.pages.dev/doom.pdf) | Doom inside a PDF file | PDFs can contain JavaScript. Chrome's PDF engine runs a very limited version of it. Doom's C source is compiled with an old Emscripten to asm.js (plain JS, no WebAssembly). The screen is one PDF text field per row, filled with ASCII characters, at about 80 ms per frame. The same author's [linuxpdf](https://github.com/ading2210/linuxpdf) boots Linux in a PDF using a RISC-V emulator. | C, Emscripten, asm.js | **Porting C to a hostile target.** You only need three things: a way to run code, input, and a framebuffer. That's the same checklist as a decomp port. |
| [NielsLeenheer/cssDOOM](https://github.com/NielsLeenheer/cssDOOM) | [cssdoom.wtf](https://cssdoom.wtf) | Doom's 3D world rendered with CSS, with no canvas and no WebGL | The level's line and sector data become `div`s with CSS custom properties (`--start-x` and so on). CSS `hypot()` and `atan2()` work out each wall's width and angle, then `transform: translate3d(...) rotateY(...)` places it. Game logic is JS written with id's open-source code as a reference. | CSS, JS | How a game's map format turns into geometry, and how to convert between coordinate systems (Doom's Y is CSS's −Z). |
| [mmulet/font-game-engine](https://github.com/mmulet/font-game-engine) | n/a | A whole game ("Fontemon") that lives inside a font file | OpenType fonts have substitution rules (ligatures) that swap glyphs depending on what you typed. Chained together, those rules become a state machine, so typing moves you through the game. | Python, Blender, OpenType | Hidden computation in data formats. "This file format is accidentally a programming language" comes up a lot in reverse engineering. |

## 3. 3D, WebGL and shaders

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [brunosimon/folio-2025](https://github.com/brunosimon/folio-2025) | [bruno-simon.com](https://bruno-simon.com) | A portfolio you explore by driving a little car around a 3D world | three.js for rendering, Rapier for physics, and a game loop written out step by step in the README (inputs → physics → vehicle → view → weather and lighting). The older, widely copied version is [brunosimon/folio-2019](https://github.com/brunosimon/folio-2019). | JS, three.js, Rapier, Vite | A real **game loop and update order**, and game feel (suspension, camera lag). Read this README before you read any decompiled main loop. |
| [PavelDoGreat/WebGL-Fluid-Simulation](https://github.com/PavelDoGreat/WebGL-Fluid-Simulation) | [live](https://paveldogreat.github.io/WebGL-Fluid-Simulation/) | Smoke-like fluid that follows your finger, even on phones | The fluid state lives in GPU textures, and small fragment shaders run each simulation step every frame. | JS, GLSL | Using the GPU for maths, not only for drawing. |
| [patriciogonzalezvivo/thebookofshaders](https://github.com/patriciogonzalezvivo/thebookofshaders) | [thebookofshaders.com](http://thebookofshaders.com) | A book whose examples are live, editable shaders | Fragment shaders: a tiny program that runs once per pixel to choose its colour. | GLSL | The basics of every modern renderer, and how to read graphics code you'll find in game binaries. |

## 4. CSS-only art

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [cyanharlow/purecss-francine](https://github.com/cyanharlow/purecss-francine) | [diana-adrianne.com/purecss-francine](https://diana-adrianne.com/purecss-francine/) | An 18th-century-style oil painting made only from HTML and CSS | Hundreds of `div`s shaped with `border-radius`, gradients, shadows and blur. Each browser draws it differently, which is part of the fun. | HTML, CSS | Layering and compositing. It also shows how "the same code" renders differently on different engines, much like emulator accuracy. |
| [lynnandtonic/a-single-div](https://github.com/lynnandtonic/a-single-div) | [a.singlediv.com](https://a.singlediv.com) | Detailed drawings that each use **one** HTML element | Each element gets two extra pseudo-elements (`::before` and `::after`), and a single element can carry many stacked gradients and shadows. | CSS (Stylus) | Doing a lot with a tiny budget, the same mindset as GBA hardware limits. |
| [MatthewRayfield/gif2css](https://github.com/MatthewRayfield/gif2css) | n/a | Turns any GIF into pure CSS | Each pixel becomes a `box-shadow` and each frame becomes a step in a `@keyframes` animation. | HTML, JS | Converting image data between formats, which is close to extracting and converting game graphics. |

## 5. Demoscene and tiny code

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [lionleaf/dwitter](https://github.com/lionleaf/dwitter) | [dwitter.net](https://www.dwitter.net) | A social network for visual demos of at most 140 characters of JS | Your code runs as the body of `u(t)` every frame, with a canvas `c`, a context `x`, and helpers `S`, `C` and `R` already provided. | Python/Django, JS | Reading extremely compressed code. Decoding a dweet feels a lot like reading optimised assembly. |
| [phoboslab/q1k3](https://github.com/phoboslab/q1k3) | [phoboslab.org/q1k3](https://phoboslab.org/q1k3/) | A Quake-like FPS in 13 KB (js13k 2021) | Generated textures, a map compiler written in C, WebGL rendering and a minifier. The [making-of article](https://phoboslab.org/log/2021/09/q1k3-making-of) is excellent. | JS, WebGL, C | How a real engine fits together when every byte counts: map formats, collision and enemy AI. |
| [KilledByAPixel/OS13k](https://github.com/KilledByAPixel/OS13k) | [live](https://killedbyapixel.github.io/OS13k/) | A fake desktop OS in a 13 KB zip | A window manager, taskbar and apps written in plain JS, with built-in support for dweets and Shadertoy shaders. | JS | Event loops and window management, with no framework to hide them. |
| [KilledByAPixel/ZzFX](https://github.com/KilledByAPixel/ZzFX) | [zzfx.3d2k.com](https://zzfx.3d2k.com) | A complete sound-effect generator in under 1 KB | Each sound is a short list of numbers (volume, frequency, slide, noise and so on) that it turns into a waveform. | JS, Web Audio | Procedural audio, close in spirit to how GBA games drive their sound channels with small parameter tables. |

## 6. Emulators and OSes in the browser

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [copy/v86](https://github.com/copy/v86) | [copy.sh/v86](https://copy.sh/v86/) | Boots Linux, Windows 98 and more in a tab | Emulates an x86 CPU and hardware. Hot code is translated to WebAssembly while it runs (JIT). | JS, Rust→Wasm | **CPU emulation and dynamic recompilation.** It's the runtime cousin of static recomp ([03](03-static-recompilation.md)). |
| [thenick775/gbajs3](https://github.com/thenick775/gbajs3) | [gba.nicholas-vancise.dev](https://gba.nicholas-vancise.dev) | A full GBA emulator in the browser | Compiles the mGBA core to WebAssembly and puts a React interface around it. You bring your own ROM. | TypeScript, Wasm | Our target platform, playable and inspectable on the web. A model for a web front end to our toolkit. |
| [caiiiycuk/js-dos](https://github.com/caiiiycuk/js-dos) | [js-dos.com](https://js-dos.com) | DOS games running on web pages | DOSBox compiled to WebAssembly, with a JS API for loading bundles. | TypeScript, C++→Wasm | Packaging an emulator so other people can build on it. |
| [ruffle-rs/ruffle](https://github.com/ruffle-rs/ruffle) | [ruffle.rs](https://ruffle.rs) | Brings dead Flash content back to life | A Flash Player rewritten in Rust and compiled to WebAssembly. It reimplements the SWF format and the ActionScript virtual machines. | Rust, Wasm | **Clean-room reimplementation** of a proprietary runtime. Compare with [08-legal-ethics.md](08-legal-ethics.md). |
| [d07RiV/diabloweb](https://github.com/d07RiV/diabloweb) | [d07riv.github.io/diabloweb](https://d07riv.github.io/diabloweb/) | Diablo 1 in a browser, even on phones | Built from the [devilution](https://github.com/diasurgical/devilution) decompilation, with its dependencies removed and its input reworked for JS, then compiled to WebAssembly. Ships the free shareware data, and you bring your own `DIABDAT.MPQ` for the full game. | C++→Wasm, JS | **Decomp → web port, end to end.** Exactly the path a finished GBA decomp could take. |
| [cloudflare/doom-wasm](https://github.com/cloudflare/doom-wasm) | [silentspacemarine.com](https://silentspacemarine.com) | Multiplayer Doom in the browser | Chocolate Doom compiled with Emscripten. Its network code is changed to use WebSockets instead of raw network packets (UDP). | C→Wasm | Swapping out one platform layer (networking) while leaving the game logic alone. |
| [1j01/98](https://github.com/1j01/98) | [98.js.org](https://98.js.org) | A Windows 98 desktop recreation | HTML and CSS recreate the look, plus JS apps (including the same author's [jspaint](https://github.com/1j01/jspaint)). | JS | Recreating a UI pixel-perfectly from reference images, a "matching" mindset for visuals. |
| [DustinBrett/daedalOS](https://github.com/DustinBrett/daedalOS) | [dustinbrett.com](https://dustinbrett.com) | A full desktop as a personal website, with a file system and apps | A browser-side file system plus embedded emulators and players. | JS/TS, Next.js | How a big front-end project is organised. |
| [captbaritone/webamp](https://github.com/captbaritone/webamp) | [webamp.org](https://webamp.org) | Winamp 2, skins included, in a browser | Reimplements the player and parses classic `.wsz` skin files (zipped bitmaps). | TypeScript, React | **Reading a legacy file format** and reproducing the original behaviour faithfully. |

## 7. Sound and interactive toys

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [googlecreativelab/chrome-music-lab](https://github.com/googlecreativelab/chrome-music-lab) | [musiclab.chromeexperiments.com](https://musiclab.chromeexperiments.com) | Google's music toys (Song Maker, Kandinsky and others) | All built with the Web Audio API. The repo is archived but readable. | JS, Web Audio | Making sound respond instantly to input. |
| [jonobr1/Patatap](https://github.com/jonobr1/Patatap) | [patatap.com](http://patatap.com) | Every key triggers a sound and a burst of animation | Each key is mapped to an audio sample and a two.js animation. | JS, two.js | **Juice**: pairing sound with motion so actions feel good. |
| [ncase/trust](https://github.com/ncase/trust) | [ncase.me/trust](https://ncase.me/trust/) | An interactive essay on game theory that you play through | Simulated agents play the prisoner's dilemma while scripted slides walk you through the ideas. A neal.fun-style "explorable" with open source. | JS, Pixi.js | Teaching through play, and agent simulation (ties into our rival-system notes). |

## 8. ASCII and terminal sites

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [ertdfgcvb/play.core](https://github.com/ertdfgcvb/play.core) | [play.ertdfgcvb.xyz](https://play.ertdfgcvb.xyz) | A live-coding playground where the "pixels" are text characters | You write a `main(coord, context)` function that returns one character per cell, the same idea as a shader but in text. | JS | Character-cell rendering, much like a GBA tile map or text layer. |
| [Cveinnt/LiveTerm](https://github.com/Cveinnt/LiveTerm) | [liveterm.vercel.app](https://liveterm.vercel.app) | Personal websites that look and work like a terminal | A Next.js app with a command parser driven by one config file. | TypeScript, Next.js | Writing a simple command interpreter, the same pattern as our `gbadt` command-line tool. |

## 9. Multiplayer weirdness

| Repo | Live | What's weird | How it works | Stack | What it teaches us |
|---|---|---|---|---|---|
| [nolenroyalty/one-million-checkboxes](https://github.com/nolenroyalty/one-million-checkboxes) | (closed; [Wikipedia](https://en.wikipedia.org/wiki/One_Million_Checkboxes)) | One million checkboxes shared by everyone in real time | The board is stored as a **bitset** (one bit per checkbox, about 125 KB). Clients get small updates over WebSockets and only draw the visible rows. The author suggests reading the launch commit first, then [the scaling blog post](https://eieio.games/essays/scaling-one-million-checkboxes/). Follow-up: [one-million-chessboards](https://github.com/nolenroyalty/one-million-chessboards). | JS, Python, Redis | Bit packing (GBA save data and flags work the same way) and keeping shared state in sync. |
| [reddit-archive/reddit-plugin-place-opensource](https://github.com/reddit-archive/reddit-plugin-place-opensource) | (event over) | Reddit's 2017 r/place: a shared canvas where each person could place one pixel every few minutes | The canvas is stored as a packed bitfield (4 bits per pixel, 16 colours) and broadcast over WebSockets, with a cooldown on each user. Archived. | Python, JS | Palettes and packed pixel formats, exactly like 4bpp GBA tiles. |

## Patterns worth remembering

1. **Find the output channel.** URL bars, favicons, PDF text fields, CSS boxes and DevTools all become screens once you can redraw them fast enough. A ported game needs the same three things: input, a framebuffer, and a way to run code.
2. **Pack data into bits.** Braille snake, one-million-checkboxes and r/place all pack pixels or states into bits. GBA graphics (4bpp and 8bpp tiles) and save flags work the same way.
3. **Compile C to the web.** Emscripten turns C or C++ into WebAssembly (or asm.js). doompdf, doom-wasm, js-dos and diabloweb all prove it, and a finished matching decomp could ship this way.
4. **Recompile while running versus beforehand.** v86 recompiles machine code while it runs. N64Recomp does it ahead of time ([03](03-static-recompilation.md)).
5. **Constraints make style.** js13k, dwitter, a-single-div and ZzFX show that tight limits lead to clever code, just like on GBA-era hardware.

## Where this fits

- Emulator and port background: [03-static-recompilation.md](03-static-recompilation.md), [06-platform-notes.md](06-platform-notes.md)
- Legal side of reimplementation and bring-your-own-data: [08-legal-ethics.md](08-legal-ethics.md)
- JS, WebAssembly and graphics stages of the study plan: [21-learning-paths.md](21-learning-paths.md)
- Developer style and engine habits: [15-developer-profiling.md](15-developer-profiling.md), [20-developer-approaches.md](20-developer-approaches.md)

No ROMs, game code or copyrighted assets are included here, only links to public repositories.
