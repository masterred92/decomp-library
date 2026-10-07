# User-Shared Media
## Atrioc clip: "Everything Is Everything Now"
- URL: https://www.youtube.com/watch?v=jp7hM1vKRpY . Channel "Big A" (@AtriocClips). Title came from YouTube oEmbed.
- **Transcript and description could not be retrieved** (YouTube returned 429 / "sign in to confirm you're not a bot" to yt-dlp). Contents are not summarised here so nothing is guessed. TODO: add notes once a transcript is available.
- Possibly related (unconfirmed): same-day news on PhotoCraft, see 11-desktop-apps-photoshop.md. Not confirmed to be the video's topic.

## Atrioc - "Everything Is Everything Now" (transcript provided by Kenny, 2026-10-07)
Summary of claims made in the video (presenter is non-technical and the claims are unverified):
- **Pass-through fusion** (most viral clips, e.g. Spider-Man over Batman, Mario in Elden Ring): two games run at once, sharing memory state (enemy position, HP, collision) and one game's character is drawn over the other's, like the Archipelago randomizers. It's mostly a visual trick; there are overlay artefacts such as the character being drawn above UI text.
- **Decomp-based mods**: the classic matching-decomp loop is to guess the source, compile it with the original compiler, diff and repeat. It used to take years. The key insight is that LLMs excel at verifiable tasks, and matching decomp has a perfect checker (does the compiled output match the bytes?), so it's now fast. See 04-decomp-tooling.md and 05-ai-llm-decompilation.md.
- Examples mentioned: original Xbox Halo CE running in a browser (said to run at 122 fps, better than the Gearbox port), multiplayer working; a claim that Windows Steam games get ported to Mac in about 2 hours.
- **Mechanic extraction and rewrites**: decompile a game, isolate a subsystem (Skate physics, Mirror's Edge parkour, early Assassin's Creed climbing), rewrite it in Rust and transplant it into another game. Skate 3's speed glitch still worked when run inside Modern Warfare 2, which suggests a faithful port of the logic.
- **Non-game software**: a decompiled Photoshop clone (he says "Photon Studio"; probably PhotoCraft, see 11-desktop-apps-photoshop.md). He found no feature differences for his thumbnail workflow. He speculates about TurboTax.
- Limits he notes: online/server-side software can't be done this way; encryption isn't broken; things that need genuinely novel reasoning are still weak spots.
- He built a Bloons-in-Age-of-Empires fusion in about 6 hours as a non-programmer.
- Takeaway for this library: the verifiable feedback loop (compile, diff, iterate) is what makes AI decompilation work, so build workflows around fast, automated match checks.
