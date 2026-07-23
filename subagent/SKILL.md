---
name: subagent
description: Include when and how to use subagents. Always invoke this skill before invoking subagents.
---

# Multi-Agent Work

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

Subagents keeps context short and clean, but requires regaining context. Always prompt the user to pick between editing inline and subagent, with your recommendation based on the task size.

## When to fan out

- **Broad exploration** — searching multiple files for bug, 
- **Fresh context needed** — reviewing code, plan; jobs that should not include bias(design decisions).
- **Long projects** — when the project is long and doing edits inline will require constant compaction.

## When NOT to fan out

- **Small tasks** — fresh new subagents will have to rediscover everything, which cost more than the benefits.
- **Anything requiring the user's live judgment** — subagents can't talk to user, pin down the design decisions with question asking skill first.

## Picking the model per agent

Match the model to the hardest thing the agent must do, not to the task's
prestige:

- **fable** — most capable and costly, only use fable when user explicitly request.
- **opus** — the default model for everything.
- **sonnet** — cheap and fast model for repetitive tasks like updating docs, filtering files.

## Reasoning level

- **fable** — always use medium unless user request other reasoning level.
- **opus** — medium for writing most cases, high for harder code writing, low for doing research.
- **sonnet** — low/medium based on task size

## No nested subagents

Subagents are never allowed to create nested subagents. All fan-out decisions are made by the orchestrating model.

## Writing the dispatch prompt

A fresh new subagent knows nothing about the conversation. The prompt must carry everything:

- **Context**: what and why — or point it at the plan file (`docs/plans/<id>-<slug>.md`).
- **Task**: clear instructions including which files it can and cannot read/write and when it should stop.
- **Constraints**: context it can't infer (or point it at the skill/reference).
- **Report format**: what to return, like what it did, what it find, or any questions it got. You are the user of subagent. Ask for facts with evidence.
- **The no-nesting rule**: state it in every dispatch explicitly — "do not spawn subagents; if the task turns out bigger than described, report that and stop."

Subagents that does not interfere, does not touch same files/folders or rely on each others' work, can be dispatch in parallel.

## Reusing subagents

Fresh new suabgent on every new task, especially code writing, will create huge cost and waste time because subagents will have to rediscover every context before they start. Reuse subagents from previous finished tasks with similar context and include what had updated to keep it update to date. Example include using subagents that wrote a custom LLM module to wire it up.

One thing that is worth using a fresh new subagent is reviewing. However, give it clear instruction(only what, no why) to review so that it can get to review asap.

## Integrating results

- **Subagent reports are claims, not evidence.** Ask the subagent to include evidence(filename, line number, link) for anything that it state
- **Double Checking.** Verify the results through testing before telling the user a milestone is done. You do not have to check after each subagent run, but check before calling it done.
- **Reporting.** Include as part of the normal summary.
