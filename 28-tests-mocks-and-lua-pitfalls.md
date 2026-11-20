# 28: Tests, mocks and Lua pitfalls

Written 2026-11-20. A follow-on to [27 §4](27-lessons-from-addon-releases.md):
a passing test is only useful if it would have failed before the fix, and if
the stand-in for the game behaves like the game.

## 1. Lua's `and` / `or` is not a ternary

Lua treats `false` and `nil` as false, and everything else as true. The
idiom `a and b or c` is not a safe "if a then b else c":

* `x and nil or y` always yields `y`, because `nil` is false, so the `or`
  branch always runs.
* `x and false or nil` always yields `nil`, for the same reason.

If a function has three possible returns (yes, no, and "I don't know"),
write a real `if` / `else`. Do not pack it into one `and` / `or` line.

## 2. Mocks have to tell the truth

A test double that is friendlier than the client will hide the bug you
care about.

* If the game hides aura or combat data during a fight, the mock must hide
  it too, by default. Only make a value readable when the client would.
* If a widget field is unset, the mock should return `nil`, not a do-nothing
  function. A dummy function makes `frame.text` look set, and later
  arithmetic or `GetText()` blows up in ways the test never sees. In dump
  scripts, read stored fields with `rawget`.
* Never call `SetText(nil)` in code that tests will touch, if the mock's
  `GetText()` then returns a function.

## 3. Render from the live state

A picture of the intended layout will not catch a logic bug. Drive a
headless render from the addon's real state and read what it actually
drew. That is how the `and` / `or` trap in §1 showed up: the mockup
looked right, and the live state put "start here" on the wrong step.

## 4. Check that the result scales

A size threshold will not catch a clamped minimum. If a fee or deposit is
supposed to grow with quantity, compare a stack of N with a stack of 2N.
If the two results are almost the same, you are looking at a floor, not a
rate. (The old "one silver" estimate sat well above a twenty-copper
posting, so a "must be larger than X" check never fired.)

## 5. Automated checkers can misread the source

A linter that looks for widget calls can be fooled by ordinary code:

* A three-argument `CreateFrame` on one line, followed by
  `CreateTexture(nil, "BACKGROUND")`, can be read as "this frame uses a
  template."
* A method you defined yourself, such as `function Row:Name(`, can be
  read as a game widget call.

Read the failure, then either adjust the source so the pattern is
unambiguous or fix the checker. Do not "satisfy" a bad match by deleting
a valid call.

## 6. Prove the test

After you add a regression test, run it against the previous commit
(for example with `git archive`) and confirm it fails there. Then restore
the fix and confirm it passes. If it passes on the broken commit, it was
not testing the bug. This is the same rule as [27 §4](27-lessons-from-addon-releases.md).

## How this evolved

- **2026-11-16.** Gale Count 1.0.0 had shipped on 12 November. A combat
  proc-tracking bug led to a first fix whose tests still fed *readable*
  aura data during a fight. The tests passed; the game did not. A review
  the same day showed the mock had to apply the client's hiding rules by
  default. Gale Count 1.0.2 went out that day with a more honest
  description of how detection works. See [30](30-hidden-data-and-addon-apis.md).
- **2026-11-20.** On NeutralFlip 0.3.3, a "deposit looks big enough"
  check missed a clamped minimum. The new test was proved by running it
  against the previous commit and watching it fail. The same day, a
  headless render of live addon state caught `x and nil or y`, and a
  mock that returned a do-nothing function for every missing field made
  a value look set when it was not.
