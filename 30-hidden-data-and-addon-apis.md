# 30: Hidden data and addon APIs

Written 2026-11-20. This goes deeper than [27 §2](27-lessons-from-addon-releases.md).
On WoW: Forever, a lot of combat information is hidden from addons while
a fight is underway. Designs and tests that assume otherwise will fail
in the game and pass in a friendly mock.

## 1. Hiding is the default in combat

Aura payloads, many aura lookups, spell-cast details and damage-meter
data are hidden during combat. "Look up this one spell I already know"
is not automatically allowed: that lookup can still return nothing
unless the spell is marked as never hidden. The combat log is not
available to addons on this client at all, so a design that waits for
combat-log events will never see them.

When a value is hidden, it is unknown. A fallback such as `x or 1` or
`x or 0` turns that unknown into a confident number. Do not do that.

## 2. Re-check when combat ends

If you need to know whether a spell is known, or to refresh something
you could not read during the fight, look again when combat ends
(`PLAYER_REGEN_ENABLED`). The event `LEARNED_SPELL_IN_TAB` does not
exist here; use `LEARNED_SPELL_IN_SKILL_LINE`, `SPELLS_CHANGED` and
`PLAYER_LEVEL_UP`. Spell trainer levels come from DB2 data, not from
memory. See [27 §2](27-lessons-from-addon-releases.md).

## 3. Pick a detection mode at runtime

Whether a given spell's aura is readable is often only knowable in the
game. Choose a mode at runtime (for example: aura if it is readable,
cast if plain cast events are seen, otherwise an honest "can't count"
note) and give each spell exactly one source. Do not invent a
time-based deduplicator to paper over two sources.

## 4. Feature-test APIs that are not in the UI source

If a function is not in the official UI source, do not assume it exists.
Feature-test it, fall back if it is missing, and record why the API
checker is allowed to pass. Keep that record next to the test so the
next person does not "fix" the checker by pretending the call is
documented.

## 5. UI that captures keys must not outlive combat

An overlay that swallows keyboard input must not still be live when a
fight starts. On this client, the call that would pass keys through is
restricted, so the safe step is to hide or disable the overlay. If you
register a frame in the game's "close this with Escape" list, do not
edit that list from the frame's own hide handler while the game is
already walking it; defer the change.

## How this evolved

- **2026-11-16.** The first Gale Count combat fix assumed that a
  per-spell aura lookup would usually be readable in a fight, because
  "individual spells may be flagged." A review the same day showed the
  opposite default: the lookup returns nothing in combat unless the
  spell is marked never hidden. The first tests had also fed plain
  data into a fight, so they could not have caught the mistake. The
  rule became: hiding is the default, mocks must do the same, and
  detection mode is chosen at runtime. Gale Count 1.0.2 shipped that
  day.
- **2026-11-20.** Reckoning Checkoning 1.0.1 started treating a hidden
  fight as "unknown" rather than a confident zero, and only counting
  spends from a visible buff. The Frost 1.0.1 review required a
  re-check of spell knowledge when combat ends, and confirmed that
  `LEARNED_SPELL_IN_TAB` is not present on this client. A review of
  Dwarf Explorer's Bible 1.2.0 added the Escape-list and keyboard
  rules in §5, and the "feature-test missing APIs" rule in §4.
