# 32: Auction-house pitfalls

Written 2026-11-20. Several of these showed up in a written review on
25 October 2026, before WoW: Forever launched, and were confirmed or
corrected in NeutralFlip 0.3.3 and Water Bill on 20 November.

## 1. Use the live deposit, and check the rate

A hardcoded cut (for example a flat five percent) and a guessed minimum
deposit will mis-price a post. Ask the auction house for the live
deposit when it is open, and treat a missing deposit as unknown, never
as zero. To see whether you are looking at a rate or a floor, compare
quantity N with 2N, as in [28 §4](28-tests-mocks-and-lua-pitfalls.md).

## 2. Listing durations are 2, 8 and 24 hours

On this client the three listing lengths are two, eight and twenty-four
hours, not twelve / twenty-four / forty-eight. Check every duration
constant and every label that says "12h". The chosen length is a saved
setting on the sell frame, not a guess.

## 3. Neutral houses need their own price bucket

A neutral auction house can use a different deposit and cut from a
faction house. If you scan there and write into the same price slot, you
overwrite faction prices. There is no dedicated "am I at a neutral
house?" API in the notes we used; detect it by zone or by the deposit
ratio, and keep a separate bucket.

## 4. Do not skip incomplete scan rows

If a full scan tells you a row is not fully loaded, request the item
data and read the row again. Skipping those rows can silently omit
items. If two of your addons walk the same scan, let one of them own
it and have the other defer.

## 5. Bound what you store

Saving every item on the auction house, plus a list of units for each,
can grow without limit on a shared ruleset. Prefer a materials list and
a watchlist, or a cap. Value a stack with a typical price minus fees
and deposit, not a single undercut.

## 6. Leave room for addons that already show a raw price

If another widely used addon already shows the cheapest buyout on a
tooltip, a second "AH price" line is noise. Spend the space on a
verdict the others do not offer (for example a break-even stack size
after fees), and say where the number came from (meter, vendor, or
auction house).

## How this evolved

- **2026-10-25.** A review of the early auction addons (the 18 October
  PriceLog, MatTooltip and FarmTracker releases in the shared
  wow-forever-addons repo) flagged a hardcoded five percent cut, a
  guessed minimum deposit, no separate bucket for neutral houses,
  skipped incomplete scan rows, unbounded saved history, two addons
  walking the same scan, and overlap with raw "AH price" lines other
  addons already show. NeutralFlip 0.1.0 followed the next day as a
  faction-versus-neutral flip finder.
- **2026-11-20.** NeutralFlip 0.3.3 and a Water Bill review showed two
  further corrections. Listing durations had been written as 12 / 24 /
  48 hours in more than one place; on this client they are 2 / 8 / 24.
  A deposit "large enough" check still missed a clamped minimum,
  because the old one-silver estimate sat above a small real deposit;
  the quantity-scaling check in §1 replaced it.
