---
name: git-habits
description: Git habits for all projects. Always invoke this before branching and committing if not in context.
---

# Git Habits

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## Ground rules

- **Never push unasked** unless permission is given explicitly by the user.
- **Commit** after you (the orchestrator) have reviewed the change, user approval isn't required.
- Ask the user what branch to work on; don't assume a new feature branch or main.
- Don't rewrite shared history — no force-push, no rebase of anything already pushed, without explicit direction.

## Commits

- **Small and focused**: one logical change per commit — reviewable and revertable in isolation. Don't bundle a refactor with a behavior change.
- Commit only what belongs in history: no generated artifacts, no secrets, no large binaries. Bundle gitignore with the commit that creates the update.
- Commit as a checkpoint after reviewing each subagent's work, so a bad result can be reverted without losing the good ones.

## Commit types

- feature: new user-visible behavior or a meaningful capability
- fix: refactor, performance improvement or bug fixes
- chore: config files, build scripts, Makefiles, CI tooling, formatters or style/linting
- test: tests, test fixtures, or test-only support code
- docs: documentation, README, examples, or comments-only changes
- dependency: package, dependency, or lockfile changes
- other: changes that do not fit the specific categories

## Commit message style

> `<type>: <summary>`

Use a concise, imperative one-line summary. Start with a lowercase word unless it is a proper name, acronym, or code identifier. Do not end with a period, and do not use past tense. When a one-line summary is not enough, split the commit.

## Gitignore: two tiers

- **Folder-local `.gitignore` containing just `*`** — co-located inside any folder whose contents shouldn't be tracked *and that readers wouldn't expect to exist* (internal `context/`, one-time scripts). The folder carries its own ignore rule, and the root file stays free of surprises.
- **Root `.gitignore`** — reserved only for *expected*-but-untracked things: artifacts any reader assumes a project of this type produces (`target/`, `node_modules/`, run output).

The test: would someone reading the root `.gitignore` learn something about the project they should know? `target/` yes; the existence of a private scratch folder, no.

## Hygiene

- `git status` before and after committing — know exactly what's staged and confirm nothing was left behind or swept in accidentally. If an unexpected file appears, read it (if it doesn't touch secrets) and then update gitignore, include it in the commit, or ignore it.

## Scratch check scripts: `./tmp/`

One-off checks or artifacts for end-to-end tests that don't belong in the test suite go in `./tmp/`, organized by the category (step, milestone, or intention — pick the most maintainable one) they belong to:

```
tmp/
├── .gitignore                    # contains "*" — nothing tracked
├── README.md                     # what each folder is for and why
├── data-generation/
│   ├── README.md                 # what each script is for and why
│   └── check_data_dist.py
└── test-environment/
    ├── README.md                 # what the environment is for and why
    └── example_puzzle.txt
```

- **Don't delete these scripts** — they're often useful again later.
- **Maintain the READMEs** (one in `tmp/`, one per category folder) and the scripts so a future subagent or session can index them quickly.
- `tmp/` carries a folder-local `.gitignore` with `*`; create it with the folder.
