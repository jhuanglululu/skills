---
name: subagent
description: Include when and how to use subagents. Always invoke this skill before invoking subagents.
---

# Multi-Agent Work

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

Subagents keep context short and clean, but require regaining context. Always prompt the user to pick between editing inline and subagent, with your recommendation based on the task size.

## When to fan out

- **Broad exploration** — finding information from multiple files or the internet
- **Long projects** — when the project is long and doing edits inline will require constant compaction.

After spawning subagents, report to the user what each subagent is for and what model + reasoning level it uses.

## When NOT to fan out

- **Small tasks** — fresh subagents have to rediscover everything, which costs more than the benefits.
- **Anything requiring the user's live judgment** — subagents can't talk to the user, so pin down the design decisions with the interviewing skill first.

## Picking the model per agent

Match the model to the hardest thing the agent must do:

| Work | Model | Effort |
|---|---|---|
| File/pattern search, inventory | sonnet | low |
| Doc updates, mechanical refactor | sonnet | low–medium |
| Research & synthesis | sonnet | medium |
| Feature implementation | opus | medium |
| Hard algorithmic / debugging | opus | high |
| Review with fresh eyes | opus | medium–high |

Only use Fable when the user explicitly requests it, and avoid haiku — sonnet at low effort costs slightly more and does better.

## How to split a task

- **Stop before a checkpoint.** When one part of a task is error-prone or requires review, like writing a custom einsum or wiring up multiple parts, split the task so that you can review before errors propagate.
- **Subagents don't decide for you.** If a task requires making a design or large implementation decision to continue working, split the task and make the decision yourself.

If a subagent reports that its task is too large, or stops before finishing, split the remaining task and use an appropriate model and reasoning effort instead of sending the same task to a larger model.

## Reusing a subagent

A fresh subagent for every new task, especially code writing, creates huge cost and wastes time because the subagent has to rediscover all the context before it starts. Reuse subagents from previously finished tasks with similar context, and tell them what has changed to keep them up to date. Example: using the subagent that wrote a custom LLM module to wire it up.

## Using a new subagent

Two circumstances worth using a subagent with fresh context are:

- **Reviewing** — when biases and assumptions can hide problems. Give it clear instructions (only what, no why) so that it can start reviewing as soon as possible.
- **The subagent's context becomes heavy** — a subagent that has been reused multiple times carries heavy context and can get compacted automatically.

Another sign to use a new subagent is when the task diverges from the one it was originally given. Example: using a new subagent to write the training loop instead of reusing the subagent that wrote the attention module.

## No nested subagents

Subagents are never allowed to create nested subagents. All fan-out decisions are made by the orchestrating model.

## Writing the dispatch prompt

A fresh subagent knows nothing about the conversation. The prompt must carry everything:

- **Context**: what and why — or point it at the plan file (`context/plans/<id>-<slug>.md`).
- **Task**: clear instructions including which files it can and cannot read/write and when it should stop.
- **Constraints**: context it can't infer (or point it at the skill/reference).
- **Report format**: what to return — what it did, what it found, or any questions it has. You are the user of the subagent. Ask for facts with evidence.
- **The no-nesting rule**: state it in every dispatch explicitly — "do not spawn subagents; if the task turns out bigger than described, report that and stop."

Subagents that do not interfere — that do not touch the same files/folders or rely on each other's work — can be dispatched in parallel.

## Integrating results

- **Subagent reports are claims, not evidence.** Ask the subagent to include evidence (filename, line number, link) for anything it states.
- **Double-checking.** Verify the results through testing before telling the user a milestone is done. You do not have to check after each subagent run, but check before calling it done.
- **Reporting.** Include as part of the normal summary.
