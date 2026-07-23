---
name: git
description: Git habits for all projects.
---

# Git

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## Ground rules

- **Never commit or push unasked** unless permission given explicitly by the user.
- **Don't rewrite shared history.** No force-push, no rebase of anything already pushed, without explicit direction.
- Always ask user what branch to work on.

## Commits

- **Small and focused**: one logical change per commit — reviewable and revertable in isolation. Don't bundle a refactor with a behavior change.
- Commit only what belongs in history: no generated artifacts, no secrets, no large binaries. Update gitignore with its own commit before the current commit.

## Commit types:

- feature: new user-visible behavior or a meaningful capability
- fix: refactor, performance improvement, bug fixes or behavior corrections
- chore: config files, build scripts, Makefiles, CI tooling, formatters or style/linting
- test: tests, test fixtures, or test-only support code
- docs: documentation, README, examples, or comments-only changes
- dependency: package, dependency, or lockfile changes
- git: gitignore, git workflows
- other: changes that do not fit the specific categories

## Commit message style:

> `<type>: <summary>`

Use a concise, imperative summary. Start with a lowercase word unless it is a proper name, acronym, or code identifier. Do not end with a period and use past tense.

## Attribution lines

- **Never include the `Claude-Session` + link line** in any commit message — the link only works for this user's account, so to everyone else reading history it's dead noise.
- **Include the `Co-Authored-By` (Claude) line only when Claude wrote 50% or more of the code in that commit.** Judge by the diff actually being committed: mostly user-written or user-dictated code → no co-author line; mostly Claude-generated → include it.

## Gitignore: two tiers

- **Folder-local `.gitignore` containing just `*`** — co-located inside any folder whose contents shouldn't be tracked *and that readers wouldn't expect to exist* (internal `docs/`, one-time scripts). The folder carries its own ignore rule, and the root file stays free of surprises.
- **Root `.gitignore`** — reserved only for *expected*-but-untracked things: artifacts any reader assumes a project of this type produces (`target/`, `node_modules/`, build output).

The test: would someone reading the root `.gitignore` learn something about the project they should know? `target/` yes; the existence of a private scratch folder, no.

## Hygiene

- `git status` before and after committing — know exactly what's staged and confirm nothing was left behind or swept in accidentally.
