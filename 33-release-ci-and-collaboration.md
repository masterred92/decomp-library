# 33: Release, CI and collaboration

Written 2026-11-20. A follow-on to [27 §6–§10](27-lessons-from-addon-releases.md)
with the smaller release and teamwork rules that are easy to skip.

## 1. Verify the live listing, not only the file

After a packager run, check `gh release view` and `gh run list`, then
open the live store summaries and read them. A wording fix in your
source does not always reach every store page. Uploads often appear in
ten to twenty minutes; a short grace period is still useful, but a wait
of that length is not, by itself, a failure. New CurseForge projects
stay hidden until moderators approve them. See
[27 §7–§8](27-lessons-from-addon-releases.md).

When you paste a description, serve the file as UTF-8 and check the
pasted length against the file. A stale clipboard, or a special
character that did not survive the paste, has produced a short or
garbled listing. Reload a store editor before you read it: it can still
be showing the previous project's text. Some "featured image" or crop
controls ignore scripted clicks and need a real mouse press; do not
navigate away while Save is still running.

## 2. Do not weaken a CI check to get a green run

If a check compares shared files to a canonical repository, and that
repository is archived (read-only), every push will fail. The safe fix
is to unarchive it, push one commit, and archive it again, then confirm
the checks are green. Fetch and fast-forward first; the remote may
already have commits you do not. See
[27 §6](27-lessons-from-addon-releases.md).

Pin third-party workflow actions that handle store credentials to a
full commit SHA, not only a moving tag. Keep internal machine paths out
of shipped files and comments; use a neutral wording or an environment
variable instead.

Do not run a scaffold script that rewrites every README and changelog
just to change one sentence. Edit the one file, and the template if you
need the next generation to match.

## 3. One writer per checkout

Do not run two editors, or a destructive clean and build, in a checkout
someone else is using. Use a separate git worktree, and look at
`git status`, `git reflog` and the remote branches before you start: the
fix you are about to begin may already have been pushed. If two people
need to change the same shared library, agree the next version number
before either of you claims it. Split work across branches only when
those branches do not overlap. See
[27 §10](27-lessons-from-addon-releases.md).

A reviewer's pass is a code review, not a release approval, and it
covers only the commits that were named. A relayed "pre-approval" is
not the owner's approval.

## How this evolved

- **2026-11-13.** Store summaries were edited so they no longer carried
  a disclaimer we had decided not to use. That was treated as done.
- **2026-11-20.** After NeutralFlip 0.3.3 went up (and on a check of
  related listings), some store summaries still had the old disclaimer.
  The rule became: re-read the live page when a release lands, not only
  the file version. Packager uploads for that release and for Reckoning
  Checkoning 1.0.1 were already visible in about eleven to eighteen
  minutes. The same day, twenty-one shared-library checks went red
  because the canonical repo was archived; the fix was one unarchive,
  one commit, and a re-archive, not a weaker check. A November 2026
  review of the public repos also asked that credential-handling
  actions be pinned to a SHA, and that internal paths stay out of
  shipped files. Featured-image and paste problems on the store sites
  showed up during the Frost gallery work later that day.
