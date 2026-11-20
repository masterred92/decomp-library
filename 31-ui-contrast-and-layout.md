# 31: UI contrast and layout

Written 2026-11-20. A follow-on to [27 §5](27-lessons-from-addon-releases.md)
on fitting text, with the contrast and layout rules that showed up while
building the Frost suite.

## 1. Measure contrast on top of the world

Additive glows sit on whatever is already on the screen. Check each
row's text against *that row's* glow, over a dark scene, a bright snowy
one, and pure white. A glow colour must not also be that row's text
colour when the glow is saturated (red on red will fail). For the Frost
rows, a stronger backdrop and a weaker glow peak was what made the
pairs pass.

A semi-transparent backdrop lets the world show through. Grey text that
is fine on a dark scene can fall below a readable contrast on snow. For
text-heavy windows, a fully opaque background is safer. A hover
highlight at full strength can wipe out grey or yellow text; measure the
worst column of the hovered row and keep the highlight mild.

## 2. Judge icons at the size they will be seen

Art that looks distinct at full size can look like a neighbour at 64 or
32 pixels. Check the small row of a contact sheet. If two icons collapse
together, give the new one a bolder outline and drop inner detail. When
you ask for a blank face, say so: a first coin icon arrived with a
currency mark on it.

A wrong texture ID can come back as a JSON error page instead of an
image; check the content type. Prefer the spell's own icon when the
addon follows one spell, rather than a generic look-alike.

## 3. Do not drop important text to avoid a collision

If a right-aligned status collides with a long title, do not hide the
status. Measure the title, a gap and the status (with the real font, not
a "six pixels per character" mock) and move the status to the first free
slot. Grow the box when you are out of combat. Hide the status only when
height is frozen and there is truly no room.

## 4. Freeze height in combat, but size it first

Boxes should not jump during a fight. Freeze their height when combat
starts, after you have sized them for the worst content they can show in
combat. Put the height formula in one helper, and do not forget the top
offset when you count lines. For a one-line label, turn wrapping off and
limit the field to one line so an ellipsis can appear; allow wrap only
on an explicit last line.

## 5. Shorten the name, not the figures

A client-side ellipsis hides whatever comes last. Never put the figures
after a long name and hope they survive. Shorten the name first, keep
the figures compact, and only then cut further if the remaining head is
still too wide. Measure with the real font's widths. See
[27 §5](27-lessons-from-addon-releases.md).

## How this evolved

- **2026-11-16.** The first mockup notes fixed fonts, backdrop pieces
  and contrast for additive glows, including a bright-world case,
  because a backdrop is not fully opaque.
- **2026-11-20.** On the Frost suite, a 0.85 backdrop with a 0.20 glow
  peak failed most accent pairs over snow; 0.92 and 0.15 passed. The
  first layout rule was "hide the status if it touches the title,"
  which silently dropped text you need (including "can't see" warnings).
  Later the same day that rule was replaced by §3: move the status, and
  hide it only when a combat-frozen height leaves no room. A review also
  caught the ellipsis rule in §5: a trailing cut hides the figures, so
  the name has to be the part that shrinks. Height bugs on the same pass
  came from a formula that ignored the top offset, which is why §4 uses
  one helper and sizes for the worst in-combat content.
