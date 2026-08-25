---
name: planning
description: How plan and design files are written. Use before writing or updating any plan or design document, when recording a decision, when resuming work that has an existing plan, or whenever the user mentions plans or designs.
---

# Plans & Designs

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

Plans and designs live in `context/plans/` and `context/designs/` — see the **context-folder** skill for the folder rules (local-only, naming, audience/format, referencing). This skill covers what goes *in* them.

## Plans (`context/plans/<id>-<slug>.md`)

```markdown
# <Title>
<timestamp>

## Goal            — one sentence, plus what "done" means
## Approach        — key design decisions and why; what you're NOT doing, reference design files if any
## Steps           — checkable list, each independently verifiable
## Notes           — anything extra that should be included (risks, additional instructions, ...)
```

Plans are context. They describe **what and why**, so a future session with no memory of this conversation can still reason about it. Short examples go in inline code blocks. Reference designs by filename where relevant; **never reference HTML examples** — they're throwaway.

Get explicit sign-off before implementing. When presenting the plan for review, **don't hand the user the markdown file** — it's agent-facing. Write a summary for review instead: directly in the response for a short plan, or a single-file HTML page for a long one. And **don't use the question tool (AskUserQuestion) for sign-off** — it times out in Claude Code while the user is still reading; ask in plain conversation and wait.

If the user redirects during implementation, update the plan file — it's the durable record of what was agreed, and the first thing to read when work resumes in a later session.

## Designs (`context/designs/`)

A design records something **decided**. Detailed what and why, with examples in code blocks. Since context/ isn't tracked by git, each design keeps its own history: a numbered list of updates at the end, each entry a brief what and why. Rewrite the body when updating — the design itself must always read as current and easy to understand; the update list is the history, the body is not.

## What not to include (plans and designs both)

Only the **what** and **why**, never the **how**.

- **No implementation code.** Only the user-facing side: feature shapes, API designs, formats.
- **No over-explanation.** Only minimal explanation; make everything else inferable from them.
- **No detailed history.** A brief what-and-why per design update is enough.
