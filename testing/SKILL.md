---
name: testing
description: When to write tests and how to write honest ones. Use before writing any test or assertion, before writing any one-off check/inspection script, or when deciding whether something needs tests at all.
---

# Testing

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## When tests apply

Writing new tests is part of **substantial** work (dev-workflow's sizing). Small tasks skip writing tests — though the existing suite still runs as verification, and if a "small" change turns out to touch bug-prone logic, that's a sign it was mis-sized.

## Risk-based: what gets a test

Write tests where a bug is **easy to introduce and hard to spot by reading** — complex functions/classes, tricky indexing or algebra (tensor shapes, einsum-style contractions), anything wrong-answer-prone.

Skip tests where **reading the code already proves correctness** — trivial glue, one-line delegations, straightforward wiring. A test there costs maintenance without buying confidence.

Glue whose correctness is only visible when run gets end-to-end verification instead of unit tests (see the verification skill).

The per-project skill defines which parts of its domain are the bug-prone ones.

## Honest tests: how expected values are built

Never build the expected value out of the same logic the code under test uses — `assert add(1, 2) == 1 + 2` just restates the implementation, so it passes even when both are wrong. In order of preference:

1. **Known answer**, hand-computed or from existing modules(when porting): `assert add(1, 2) == 3`, `assert add(1, 2) == correct_add(1, 2)`
2. **Round-trip**, when an inverse exists: `assert sub(add(1, 2), 2) == 1`
3. **Behavior/property**, when exact values are impractical: `assert is_int(add(1, 2))`, shape/dtype/invariant checks

If you catch yourself importing or re-deriving the implementation's formula inside the test, stop — the test is either redundant or too complex, skip the test or use 2. or 3.

## Scratch check scripts: `./tmp/`

One-off checks that don't belong in the test suite (inspecting a distribution, reading at intermediate output) go in `./tmp/`, organized by the step or milestone they belong to:

```
tmp/
├── .gitignore                    # contains "*" — nothing tracked
├── README.md                     # what and why each folder is for
└── data-generation/
    ├── README.md                 # what and why each script is for
    └── check_data_dist.py
```

- **Don't delete these scripts** — they're often useful again later.
- **Maintain the READMEs** (one in `tmp/`, one per step folder) with the and reuse them without re-reading every script. what and why of each folder/file, so a future step or milestone can find
- **Scripts freeze when their step/milestone completes.** If a later step needs a script from an earlier one, copy it into the current step's folder and edit the copy — never edit the frozen original. The old script must keep working exactly as it did for its own step.
- `tmp/` carries a folder-local `.gitignore` with `*` (per the git skill's two-tier rule); create it with the folder.
