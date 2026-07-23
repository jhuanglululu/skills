---
name: verification
description: Evidence rules for claiming work is done. Use BEFORE claiming anything is done, fixed, complete, or passing; before presenting any finished work to the user; and before committing.
---

# Verification & Review

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## Evidence rules

- Never claim work is complete, fixed, or passing without having run the thing that proves it.
- State how many passed, skipped and failed.

## The ladder (general shape)

1. **Static checks** — the project's formatter and linter, clean.
2. **Unit tests** — the full relevant suite, output shown. Whether the change also deserves *new* tests is defined in the testing skill.
3. **Run it for real** (Optional) — end-to-end on real (small) input and look at the output.

The project-type skill defines the concrete verification setup.

## Self-review before presenting

Before presenting any work — small tasks included — list and double check the files edited. Report any unexpected/missing file change.

## Presentation format

Match the format to the size of what's being presented:

- **Short and simple** (a small diff, a passing test run, a one-paragraph summary): plain response in the conversation. Don't dress it up.
- **Too long to read comfortably in chat** (multi-part results, many files changed, output worth comparing side by side): a **single-file HTML** page — it's user-facing, so the format rule from the planning skill applies: show with examples and real output, not walls of prose.
