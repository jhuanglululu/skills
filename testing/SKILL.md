---
name: test-audit
description: Invoke whenever writing, changing, reviewing, or sweeping tests. Authoring gate for new tests plus audit workflow for low-value, implementation-coupled, or duplicative tests and the test-only production seams they demand.
---

# Testing

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

Two modes, one value bar. Authoring mode gates every new or changed test at write time. Audit mode runs focused sweeps of tests that re-assert source, duplicate stronger proof, couple behavior to implementation, or keep test-only production seams alive.

## Rule of thumbs

Only write tests for **behaviors** and hard to verify codes. Most bugs can't be caught by unit tests. Verifying if something works after each checkpoint should be the main test, not unit tests.

Never write tests:
- that will break when "the implementation changes, but not the behavior"
- when you can prove it correct by reading code
- for a feature that is more direct to test in by trying
- for test converage
- because you think there should be a test

Test requires maintainance and cost more when it provides nothing useful.

## What tests are for

**Tests are for implementation assumptions.** Never rely on tests to prove whether a program or feature is correct. Tests are only for avoiding mistakes, not for repeating code. Most code doesn't need tests at all — subagent review and end-to-end tests (see the verification skill) can find more bugs than tests.

## Authoring gate

Before adding any test, answer four questions; a missing answer means do not add it yet:

1. What observable behavior, invariant, or independent contract does it protect?
2. What credible regression makes it fail?
3. Why does existing coverage not already catch that failure? Each contract has one primary test owner at the strongest boundary; another layer needs its own distinct risk, such as a transport or lifecycle failure the owner cannot reach. Prefer extending a table-driven case or shared fixture over a near-duplicate test; consolidate duplicated setup in the same change.
4. Does it need a production seam (export, flag, wrapper, injection hook) that no production caller needs? If yes, move the test to the real boundary instead.

Then check the test against every [junk pattern](#junk-patterns); a match fails the gate unless the [retention bar](#retention-bar) names the contract it independently guards. A test that would break under behavior-preserving refactoring is asserting implementation, not behavior; rewrite it at the owning boundary before landing it.

Bug regression tests must fail on the pre-fix code for the intended reason and pass after the owner-boundary repair. A regression test that never demonstrably failed proves the mock, not the fix. One regression at the owner boundary covers the bug; do not replay the same scenario at every layer it crosses.

## Junk patterns

The shared checklist for both modes: the authoring gate rejects a new test that matches one, and audits hunt for existing tests that do.

- assertion-free coverage probes;
- self-comparisons and identity copiers;
- copied fixtures, inventories, manifests, or export lists;
- exact source, import, or string greps;
- private predicate or call-shape tests duplicated at real boundaries;
- duplicate invocations of the same contract;
- provider-local replays of shared helpers;
- tests whose only purpose is preserving test-only exports, globals, or wrappers;
- dead production code whose only callers are tests;
- expected values produced by the helper or renderer under test;
- mocks that implement the asserted behavior, or one identical mock standing in for different APIs;
- fixtures that supply the receipt, admission, or callback ordering the owner should produce, or persistence asserted against a store the path never writes;
- capability tests that restate declared flags instead of exercising the delivery or acknowledgement the flag promises;
- negative controls that pass for an unrelated reason, such as a denial from a different guard or a rejection the production path never reaches;
- names or fixtures that promise more than the input exercises, such as a "retires the window" test asserting the window was not cleared.

## Honest tests

If you ever find yourself needing to write a test, **never** build the expected value out of the same logic the code under test uses — `assert add(1, 2) == 1 + 2` just restates the implementation, so it passes even when both are wrong. Ways to write tests, in order of preference:

1. **Known answer**, hand-computed: `assert add(1, 2) == 3`
2. **Reference answer**, from existing modules (when porting): `assert add(1, 2) == correct_add(1, 2)`
3. **Round-trip**, when an inverse exists: `assert sub(add(1, 2), 2) == 1`
4. **Behavior/property**, when exact values are impractical: `assert is_int(add(1, 2))`, shape/dtype/invariant checks

If you catch yourself importing or re-deriving the implementation's formula inside the test, stop — the test is either redundant (in most cases) or too complex; skip the test or use 3 or 4.

## Value bar

Tests justify their maintenance cost by protecting behavior, a credible regression, or an independently meaningful contract. In an audit, an existing test that must change for behavior-preserving source reorganization is suspect, not automatically deletable; the authoring gate still rejects new ones.

Before judging a candidate, read the complete test and production owner, its entry point, callers, callees, sibling implementations, overlapping tests, CI routing, and relevant history. Read root and scoped `AGENTS.md` files first. When the test claims dependency-backed behavior, inspect the dependency source or types directly.
