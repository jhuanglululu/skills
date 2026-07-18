---
name: verification
description: Evidence rules for claiming work is done — the verification ladder (static checks → tests → run it for real), pasting actual output instead of summaries, stating skipped rungs, self-reviewing the full diff before presenting, and picking plain-text vs single-file-HTML presentation by result size. Use BEFORE claiming anything is done, fixed, complete, or passing; before presenting any finished work to the user; and before committing.
---

# Verification & Review

General guidance, not law: when the user's prompt says otherwise, the
prompt wins.

## Evidence rules

- Never claim work is complete, fixed, or passing without having run the
  thing that proves it — and **paste the actual output**, not a summary of
  what you expect it to say.
- If something failed or was skipped, say so plainly. "Tests pass except X"
  is honest; "done" with a silent skip is not.
- Verification climbs a ladder — cheap checks first, expensive proof last.
  How far to climb depends on what the change touches. **State which rungs
  you skipped and why** rather than silently stopping early.

## The ladder (general shape)

1. **Static checks** — the project's formatter and linter, clean.
2. **Unit tests** — the full relevant suite, output shown. Running existing
   tests applies to every task size; whether the change also deserves *new*
   tests is defined in the testing skill.
3. **Run it for real** — exercise the changed behavior end-to-end on real
   (small) input and look at the output. Tests passing is not the same as
   the feature working.

The project-type skill defines the concrete rungs — the exact commands, what
a real run means in its domain, any domain-specific rungs above these, and
which projects genuinely can't climb past a given rung.

## Self-review before presenting

Before presenting any work — small tasks included — re-read the **complete
diff** with fresh eyes, as if reviewing a stranger's PR:

- Does each change trace back to the goal? Delete anything that doesn't
  (debug prints, commented-out code, drive-by refactors nobody asked for).
- Could any change break behavior *outside* the diff — callers, configs,
  defaults someone else depends on?
- Hunt the domain's silent killers — the project-type skill supplies that
  checklist.

Fix what you find, re-run the affected rungs, then present: what changed,
which rungs ran, and their actual output.

## Presentation format

Match the format to the size of what's being presented:

- **Short and simple** (a small diff, a passing test run, a one-paragraph
  summary): plain response in the conversation. Don't dress it up.
- **Too long to read comfortably in chat** (multi-part results, many files
  changed, output worth comparing side by side): a **single-file HTML**
  page — it's user-facing, so the format rule from the planning skill
  applies: show with examples and real output, not walls of prose.
