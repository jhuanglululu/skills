# Git

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
