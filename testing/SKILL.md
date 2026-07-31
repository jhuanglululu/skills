---
name: testing
description: When to write tests and how to write honest ones. Use before writing any test or assertion, before writing any one-off check/inspection script, or when deciding whether something needs tests at all.
---

# Testing

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## What tests are for

**Tests are for implementation assumptions.** Never rely on tests to prove whether a program or feature is correct. Tests are only for avoiding mistakes, not for repeating code. Most code doesn't need tests at all — subagent review and end-to-end tests (see the verification skill) can find more bugs than tests.

## Risk-based: what gets a test

Write tests where a bug is hard to find by reading, like inside complex logic or tricky math. Skip tests where the code already proves correctness. Adding unnecessary tests costs maintenance without buying confidence.

The per-project skill defines which parts of its domain are the bug-prone ones.

## Honest tests: how expected values are built

Never build the expected value out of the same logic the code under test uses — `assert add(1, 2) == 1 + 2` just restates the implementation, so it passes even when both are wrong. Ways to write tests, in order of preference:

1. **Known answer**, hand-computed: `assert add(1, 2) == 3`
2. **Reference answer**, from existing modules (when porting): `assert add(1, 2) == correct_add(1, 2)`
3. **Round-trip**, when an inverse exists: `assert sub(add(1, 2), 2) == 1`
4. **Behavior/property**, when exact values are impractical: `assert is_int(add(1, 2))`, shape/dtype/invariant checks

If you catch yourself importing or re-deriving the implementation's formula inside the test, stop — the test is either redundant (in most cases) or too complex; skip the test or use 3 or 4.
