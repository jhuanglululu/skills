---
name: dev-workflow
description: The core development FLOW for all projects — task sizing (small vs substantial) and the phase order that ties the companion skills together (interview → plan → implement → debug → verify → review → git). Use at the start of any development task (feature, bugfix, refactor, investigation) in any language or project type. Always use together with the matching project-type skill (e.g. python-llm-training, rust-cli) when one exists, and invoke the companion skill for each phase as it arrives.
---

# Dev Workflow

The flow that ties the process skills together. This skill owns *when* each
thing happens; *how* to do each thing lives in its own skill (interview,
planning, testing, debugging, verification, git, multiagent) — invoke them
as their phase arrives.

This skill is general guidance, not law: when the user's prompt says
otherwise, the prompt wins — follow it without asking the skill's
permission.

## How this layers

- **Companion skills** (interview, planning, testing, debugging,
  verification, git, multiagent) define *how* to do each part of the
  process. They're standalone — usable without this skill — but this skill
  defines when each one enters.
- **Project-type skills** (e.g. `python-llm-training`, `rust-cli`) define
  *domain content*: what questions matter, what risks to look for, what
  verification proves, which tools to run. When a project-type skill says
  "the smoke run is rung 3 of the ladder," the ladder itself is the
  verification skill.

## The flow

### 1. Size the task

Every task, regardless of size, gets an interview (when anything is
unclear), sign-off before code, and review after — sizing changes how
heavyweight each is, and whether tests and a plan file are involved. How to
ask is the **interview** skill.

- **Small** (a mistake costs minutes: config tweak, bugfix with known cause,
  small refactor, throwaway script): clarify what's unclear, state your
  intended change in a sentence or two, and get a quick go-ahead, then code.
  No plan file, and no new tests (per the **testing** skill). After:
  present the diff for review — small doesn't mean unreviewed.
- **Substantial** (a mistake costs more than an hour of human or machine
  time; anything multi-step or design-shaped): interview → plan file →
  sign-off → code, with tests per the **testing** skill. Plan file format:
  the **planning** skill.

Project-type skills may define additional task types with their own process.

### 2. Implement

General rules only — domain rules live in the project-type skill:

- Match the existing codebase's conventions, style, and structure. Don't
  reorganize a repo to fit a template.
- If you add an import, add the dependency to the project's manifest in the
  same change, using the project's package manager.
- **Testing:** invoke the **testing** skill before writing any test.
- **Delegating:** if the work splits into independent pieces or needs broad
  exploration, prompt the user to pick between inline and subagents — with
  your recommendation based on the size of the task — and invoke the
  **multiagent** skill before dispatching any.

### 3. Debug

Any bug, test failure, or "this should be working" moment: stop and invoke
the **debugging** skill **before proposing a fix**. No fixes before a
diagnosis.

### 4. Verify before claiming done

"Done" is a claim about evidence, not intention. Invoke the
**verification** skill before claiming any task is complete. Minimum,
always: the project's linter/formatter is clean and relevant tests ran, with
real output shown.

### 5. Review

Before presenting work of any size, re-read the full diff with fresh eyes —
a small diff takes seconds to re-read, so this rung is never skipped. The
method is in the **verification** skill; the project-type skill supplies
the domain-specific checklist of silent killers to hunt for.

### 6. Git

Invoke the **git** skill when branching, committing, or preparing to share
work.

## Companion skills

| Skill | Invoke it when |
|---|---|
| `interview` | Anything about the task is unclear — any size, before asking |
| `planning` | Task is substantial — before writing the plan file |
| `testing` | Before writing any test or scratch check script |
| `multiagent` | Work could split across subagents — before dispatching any |
| `debugging` | Anything behaves unexpectedly — before proposing fixes |
| `verification` | Before claiming any task is done; before presenting work |
| `git` | Branching, committing, or sharing work |
