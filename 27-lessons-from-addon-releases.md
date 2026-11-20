# 27: Lessons from addon releases (November 2026)

Written 2026-11-20 after shipping the Frost addon suite (version 1.0.1)
and related store and CI work. These are general engineering lessons.
There is no game code or game assets here.

## 1. Shared library versioning

Several addons can load the same shared library, and the copy with the
highest MINOR number wins. If you change the library's content, you must
bump MINOR and add a new hash line. Never rewrite a released or recorded
hash.

If two different bodies share the same MINOR, the game picks whichever
loads first, silently. A check that only compares the current file to its
current hash will miss this. The useful check is: old hash lines stay
immutable, and a content change that does not bump MINOR fails the build.

## 2. Client restrictions

On WoW: Forever, aura and combat-log data are hidden while you are in
combat. If you need to know whether a character has learned a spell,
re-check when combat ends (`PLAYER_REGEN_ENABLED`).

The event `LEARNED_SPELL_IN_TAB` does not exist on this client. Use
`LEARNED_SPELL_IN_SKILL_LINE`, `SPELLS_CHANGED` and `PLAYER_LEVEL_UP`
instead. Take spell trainer levels from DB2 data (for example wago.tools),
not from memory.

## 3. Low-level UX

Addons that depend on a spell should still be testable at level 1. Show
a short "you learn this at level N" state, with a "Hide until I learn"
button, visible by default.

Save that hide choice per character. If the character name is unreadable
(the client can hide it), keep the hide for the current session only and
do not write it to saved settings. Otherwise you can store the wrong
character's choice, or write an unreadable key.

## 4. A test you can prove

A passing test is weaker than a test you can break on purpose. After you
add a regression test, temporarily revert the fix and confirm the test
fails. If it still passes, it was not testing the bug.

## 5. Fitting text into a fixed width

When a row has a name and some figures, shorten the name first, then the
figures. Handle the last-resort case: if the remaining head is still
wider than the box, cut it (and account for colour codes and multi-byte
characters). A comment that says "we never cut the figures" will go stale
the moment you add that last resort.

## 6. CI against an archived repo

A check that compares shared files to a canonical repo will fail on every
push if that repo is archived (read-only). The safe fix is not to weaken
the check. Unarchive the repo, push one commit, then archive it again.
Verify the checks go green afterwards.

## 7. Release flow

These addons ship with the BigWigs packager, which runs on `v*` tags and
uploads to CurseForge and Wago. After tagging, verify with
`gh release view` and `gh run list`. New CurseForge projects stay hidden
until moderators approve them, even if the upload succeeded.

## 8. Store hygiene

Never put the word "untested" on a listing, in a summary, or in a
changelog. Label generated images as Mockup and real screenshots as
in-game, and keep both if you have them. After a release, re-read the
live store summaries, not just the file versions. Featured images should
be consistent across a suite.

## 9. Personal data in public repos

Before a public push, scan the new files, the commit message and the
current tree for personal information and credentials. Also scan git
history: a name removed from today's README can still appear in an old
commit. A surname in an intro is easy to miss if you only search the
working tree.

Do not rewrite public history to hide a low-risk old line unless the
owner asks for that. For new commits, write the message in a file
(`git commit -F`) so a failed command does not get pasted into the log.

## 10. Team workflow

Specialist teammates plus an independent reviewer caught real blockers,
including the shared-library MINOR mistake that the usual lib check
missed. A reviewer's pass is a code review, not a release approval.

Do not split work across people when they would edit the same branches;
that mostly causes merge clashes. Split only when the branches do not
overlap. A relayed "pre-approval" is not the owner's approval.
