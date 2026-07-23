---
name: debugging
description: Systematic debugging discipline, invoke this skill before starting any debugging.
---

# Systematic Debugging

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

General discipline: **reproduce, analyze, hypothesize, test** before any diagnosis and fix.

## The investigation order

1. **Reproduce.** Recreate the bug and target the root cause by shrinking the scale. Skip this step if the bug caused machine/system failure. Reproduce safely.
2. **Analyze.** Find traces and hints that point to the cause. Support your claim with evidence.
3. **Hypothesize.** Create a what and why for the bug. Then propose it to the user. You and the user don't interact with the project the same way, so you or the user might missed something.
4. **Test.** Test the hypothesis directly (skip if it cause machine/system failure).
6. **Fix.** Apply the fix and verify(with **verification** skill) and ask user to review.
7.
