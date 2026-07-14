# Multi-Agent Work

Subagents trade context for coordination: each one starts fresh, works in
isolation, and returns only its final report. Used well, they parallelize
independent work and keep the main conversation's context clean. Used badly,
they duplicate work, lose the user's intent in translation, and return
confident summaries of things they misunderstood.

## When to fan out

- **Independent subtasks** — pieces that share no state and don't depend on
  each other's results. If the plan's Steps are truly "each independently
  verifiable" (per the planning reference), some of them can be dispatched.
- **Broad exploration** — searching a large codebase, sweeping many files
  for a pattern, surveying options. The subagent reads a hundred files so
  the main context doesn't have to; only the conclusion comes back.
- **Independent verification** — reviewing a diff, adversarially checking a
  claim or a suspicious finding. A fresh-context agent has no attachment to
  the work it's reviewing, which is exactly what a reviewer needs.

## When NOT to fan out

- **Sequential dependencies** — if step B needs step A's output, dispatching
  them together just makes B guess. Chain them or do them inline.
- **Shared mutable state** — two agents editing the same files produce
  conflicts or silent overwrites. Split by file/module boundary, or use
  isolated worktrees, or don't parallelize.
- **Small tasks** — dispatch overhead (writing the prompt, reading the
  report, correcting misunderstandings) can exceed doing it directly.
- **Anything requiring the user's live judgment** — subagents can't ask the
  user questions. Interview first (per the interview reference), dispatch
  after direction is settled.

## Picking the model per agent

Match the model to the hardest thing the agent must do, not to the task's
prestige:

- **fable** — genuinely hard code and finding tricky bugs. The most capable
  tier, with usage to match: reserve it for work where the others would
  plausibly fail, not for volume.
- **opus** — the default workhorse: normal code writing, analyzing larger
  files or several files at once. Smart at reasonable usage.
- **sonnet** — never for writing code. Analysis of small files only:
  summarizing, extracting, cheap read-and-report legs of a fan-out.

**Reasoning level: medium or high (prefer high). Nothing else** — lower
tiers cut exactly the deliberation subagents need to work unsupervised, and
higher ones aren't worth the spend for delegated pieces.

## No nested subagents

Subagents must not spin up subagents of their own. All fan-out decisions —
what to dispatch, when to continue, when to stop — belong to the main
model, so the user can monitor the tree from one place and steer it. If a
subagent's report shows its task was bigger than expected, it should say so
and return; the main model decides whether to dispatch the follow-up.

## Writing the dispatch prompt

A subagent knows nothing about this conversation. The prompt must carry
everything:

- **Context**: what the project is, what's being built and why — or better,
  point it at the plan file (`docs/plans/<id>-<slug>.md`), which is written
  exactly for context-less readers.
- **Task**: one clearly bounded piece of work, with explicit boundaries —
  which files/dirs it owns, and what it must NOT touch.
- **Constraints that apply**: the relevant project conventions it can't
  infer (or point it at the skill/reference that defines them).
- **Report format**: what to return — findings, file paths, decisions made,
  anything it was unsure about. Ask for facts, not vibes ("list the call
  sites found" beats "investigate the usage").
- **The no-nesting rule**: state it in every dispatch — "do not spawn
  subagents; if the task turns out bigger than described, report that and
  stop." The rule only binds if the agent is told.

Dispatch genuinely independent agents in a single round so they run
concurrently.

## Integrating results

- **Subagent reports are claims, not evidence.** Verification rules from the
  verification reference apply: anything an agent says works must be backed
  by output — rerun the relevant rungs yourself, or require the agent to
  include the actual output in its report.
- **Merge deliberately.** After parallel edits, re-read the combined diff as
  a whole (self-review rules apply to the merged result, not just each
  piece) — integration seams are where parallel work breaks.
- **Report what agents did to the user** as part of the normal summary —
  which pieces were delegated, what came back, what you verified.
