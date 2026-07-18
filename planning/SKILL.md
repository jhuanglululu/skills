---
name: planning
description: How plan and design files are organized — the local-only docs/ directory (own "*" .gitignore, never committed), plan files as docs/plans/<id>-<slug>.md with sign-off before code, design records in docs/designs/ with numbered update logs, and the user-facing-HTML vs agent-facing-markdown format rule. Use before writing or updating any plan or design document, when recording a decision, when resuming work that has an existing plan, or whenever the user mentions plans, designs, or the docs folder.
---

# The docs/ Directory — Plans & Designs

General guidance, not law: when the user's prompt says otherwise, the
prompt wins. (How to interview before writing a plan is the interview
skill.)

All design files live in `docs/`. It belongs to the user and the agents
working on the project — **never committed**. It contains its own
`.gitignore` with just `*`; create that file when creating `docs/` (see the
git skill for the gitignore rules).

```
docs/
├── .gitignore     # contains "*" — nothing in docs/ is tracked
├── examples/      # longer examples shown while interviewing/proposing (see interview skill)
├── plans/         # plans saved for other sessions
└── designs/       # things that were decided
```

## Format rule: pick by audience

- **User-facing → single-file HTML** (`examples/`): explain with *examples*
  instead of words and sentences — that's the whole reason for HTML over
  markdown.
- **Agent-facing → markdown** (`plans/`, `designs/`): explain clearly with
  words so an agent without this conversation's context can understand
  easily.

## Plans (`docs/plans/<id>-<slug>.md`)

`<id>` is the next sequential number, so the user can refer to "plan 3" in
conversation. Timestamp goes on a line under the title.

```markdown
# <Title>
<timestamp>

## Goal            — one sentence, plus what "done" means
## Approach        — key design decisions and why; what you're NOT doing
## Steps           — checkable list, each independently verifiable
## Risks           — what could silently go wrong, and the check that catches it
```

A plan must carry its own context: a description plus the top-level **what
and why**, written so a future session with no memory of this conversation
can still reason about it. Short examples go in inline code blocks.
Reference designs by filename where relevant; **never reference examples** —
they're throwaway.

Get explicit sign-off before implementing. When presenting the plan for
review, **don't hand the user the markdown file** — it's agent-facing.
Write a summary for review instead: directly in the response for a short
plan, or a single-file HTML page for a long one (the interview skill's
examples format). And **don't use the question tool (AskUserQuestion) for
sign-off** — it times out in Claude Code while the user is still reading;
ask in plain conversation and wait.

If the user redirects during implementation, update the plan file — it's
the durable record of what was agreed, and the first thing to read when
work resumes in a later session.

## Designs (`docs/designs/`)

A design records something **decided**. Detailed what and why, with examples
in code blocks. Since docs/ isn't tracked by git, each design keeps its own
history: a numbered list of updates at the end, each entry a brief what and
why. Rewrite the body when updating — the design itself must always read as
current and easy to understand; the update list is the history, the body is
not.

## What not to include (plans and designs both)

- **No implementation code.** Only the user-facing side: feature shapes, API
  designs, formats — the *what*, never the *how to write it*.
- **No over-explanation.** State the what and the why; make everything else
  inferable from them.
- **No detailed history.** A brief what-and-why per design update is enough.
