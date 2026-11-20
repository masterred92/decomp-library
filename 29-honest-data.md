# 29: Honest data — sources, rebuilds and "looks right"

Written 2026-11-20. Numbers that look tidy are not the same as numbers
you can defend. This chapter is about citing, rebuilding and checking
totals, whether the subject is a policy table or a game data file.

## 1. A screen that looks right is not a test

If a control is supposed to change a result, the test must move the
control and check that the output changes. Rendering the page, or
checking that a default figure is present, is not enough. A cap or lever
that is applied in one mode must be checked in the others too. A test
that matches a number with a regular expression can stop guarding that
number the moment the text moves.

## 2. Link each figure to the publisher who issued it

Name the source that actually published the number, not a neighbour of
it. Before you write that one event caused another, check the clocks:
a Friday report cannot explain a Thursday move. Check the direction of
a comparison ("above" or "narrows the gap") against the two rates you
just quoted. If a claim conflicts with a record you already verified,
drop the claim.

## 3. Totals must match their parts

Build a total from the same central estimates you used in the breakdown.
Then add the parts again before you publish. Mixing a low-end figure
into the total while the breakdown uses the central one will make the
other items look smaller than they are.

## 4. Rebuild the last release before you trust a new one

When a data file is generated from downloaded tables, keep the exact
inputs (the tables, the hotfix file, the checksums and the command).
Before you read a new diff, regenerate the previous release from those
saved inputs and byte-compare it with what you shipped. If that rebuild
does not match, your tooling changed, and the new diff is not meaningful
yet.

## 5. New tables can arrive before their hotfixes

A new client build can show up in a data browser before that build's
hotfix list does. If the new tables are identical to the previous
build's, generate the file with the previous build's hotfixes and say so
in the header. Otherwise you can ship a file that silently drops
hotfix-only names.

Some official hotfixes never appear in the public client tables at all
(they may exist only on the server). Record those as sourced overrides,
with a link, and flag any number the official note did not publish.
Do not invent the missing number.

## 6. Do not guess from the "obvious" table

A "recent activity" rule needs a whitelist from the data tables, not
"any action the player took." Rank values for a talent may live in a
curve table, not in the spell-effect table you looked at first. An item
that looks free to a vendor can still have a sell price; read the price
field instead of assuming zero.

## How this evolved

- **2026-11-03.** The Policy Simulator first appeared, and the first
  accuracy note was that a lot of it "just looks right initially":
  controls did not change outputs. The rule became: test that inputs
  move outputs, not only that the page renders.
- **2026-11-05 to 2026-11-09.** Briefing drafts linked a figure to the
  wrong publisher, blamed an event that happened after the move it was
  meant to explain, reversed a rate-gap direction, and once used a
  claim that conflicted with a hold already on the record (28 October).
  On 9 November a tax-lever total used a low-end fuel figure while the
  breakdown used the central estimate. The rules in §2 and §3 come from
  those corrections.
- **2026-11-16.** A review of a simulator data refresh found a cap
  applied in one mode but not another, and tests that stopped guarding a
  value once the wording moved.
- **2026-11-20.** While rebuilding Dwarf Explorer's Bible 1.2.0, the
  pipeline was checked by regenerating the previous release and
  byte-comparing it. That afternoon we also learned that a new client
  build can land before its hotfix list: if the tables match the last
  build, keep the old hotfixes rather than drop names. A 19 November
  hotfix that never reached the public tables is now stored only as a
  sourced override, with unpublished numbers flagged instead of filled
  in.
