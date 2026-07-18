---
name: git
description: Git habits for all projects — never commit or push unasked, feature branches for substantial work, small focused commits, running `mygit commit rule` for the current commit message format, and the two-tier gitignore rule (folder-local "*" files vs root .gitignore). Use when branching, committing, writing or editing any .gitignore, preparing work to share, or whenever the user asks for a commit.
---

# Git

General guidance, not law: when the user's prompt says otherwise, the
prompt wins.

## Ground rules

- **Never commit or push unasked.** Prepare the change, show what would be
  committed; the user pulls the trigger unless they've told you to commit as
  you go.
- **Don't rewrite shared history.** No force-push, no rebase of anything
  already pushed, without explicit direction.
- Substantial work happens on a **feature branch**, named `<type>/<slug>`
  (e.g. `feat/lora-support`, `fix/nan-loss`). Small fixes on the current
  branch are fine.

## Commits

- **Small and focused**: one logical change per commit — reviewable and
  revertable in isolation. Don't bundle a refactor with a behavior change.
- **Message format**: run `mygit commit rule` with bash and follow what it
  prints — it's the up-to-date source of truth for commit message
  formatting. Run it fresh rather than relying on a remembered format; the
  rules can change.
- Commit only what belongs in history: no generated artifacts, no secrets,
  no large binaries. If the current gitignore setup doesn't cover an
  artifact a new change produces, fix that in the same commit — using the
  right tier below.

## Attribution lines

- **Never include the `Claude-Session` + link line** in any commit
  message — the link only works for this user's account, so to everyone
  else reading history it's dead noise.
- **Include the `Co-Authored-By` (Claude) line only when Claude wrote 50%
  or more of the code in that commit.** Judge by the diff actually being
  committed: mostly user-written or user-dictated code → no co-author
  line; mostly Claude-generated → include it.

## Gitignore: two tiers

- **Folder-local `.gitignore` containing just `*`** — co-located inside any
  folder whose contents shouldn't be tracked *and that readers wouldn't
  expect to exist* (internal `docs/`, one-time scripts). The folder carries
  its own ignore rule, and the root file stays free of surprises.
- **Root `.gitignore`** — reserved only for *expected*-but-untracked things:
  artifacts any reader assumes a project of this type produces (`target/`,
  `node_modules/`, build output).

The test: would someone reading the root `.gitignore` learn something about
the project they should know? `target/` yes; the existence of a private
scratch folder, no.

## Hygiene

- `git status` before and after committing — know exactly what's staged and
  confirm nothing was left behind or swept in accidentally.
