---
name: dev-workflow
description: Core development workflow for ALL projects — task sizing, when and how to ask questions, how plan files are organized, systematic debugging discipline, verification evidence rules, and git habits. Use at the start of any development task (feature, bugfix, refactor, investigation) in any language or project type. Always use together with the matching project-type skill (e.g. python-llm-training) when one exists: this skill owns the process, the project-type skill adds domain specifics.
---

# Dev Workflow

Process rules that apply to every project, regardless of language or domain.

This skill is general guidance, not law: when the user's prompt says
otherwise, the prompt wins — follow it without asking the skill's
permission.

## How this layers with project-type skills

Project-type skills (e.g. `python-llm-training`) define *domain content*: what
questions matter, what risks to look for, what verification proves, which
tools to run. This skill defines *process*: when to plan, how to ask, how to
debug, what "done" means. The two never repeat each other — when a
project-type skill says "the smoke run is rung 3 of the ladder," the ladder
itself is defined here. If both are relevant, both apply.

## Workflow

### 1. Size the task

Every task, regardless of size, gets an interview (when anything is
unclear), sign-off before code, and review after — sizing changes how
heavyweight each is, and whether tests and a plan file are involved. How to
ask is in `references/interview.md`.

- **Small** (a mistake costs minutes: config tweak, bugfix with known cause,
  small refactor, throwaway script): clarify what's unclear, state your
  intended change in a sentence or two, and get a quick go-ahead, then code.
  No plan file, and no new tests (see `references/testing.md`). After:
  present the diff for review — small doesn't mean unreviewed.
- **Substantial** (a mistake costs more than an hour of human or machine
  time; anything multi-step or design-shaped): interview → plan file →
  sign-off → code, with tests per `references/testing.md`. Plan file format:
  `references/planning.md`.

Project-type skills may define additional task types with their own process
(e.g. experiments in training projects).

### 2. Implement

General rules only — domain rules live in the project-type skill:

- Match the existing codebase's conventions, style, and structure. Don't
  reorganize a repo to fit a template.
- If you add an import, add the dependency to the project's manifest in the
  same change, using the project's package manager.
- **Testing:** read `references/testing.md` before writing any test — it
  defines what deserves a test (risk-based) and how expected values must be
  built (honest tests, never restating the implementation's logic).
- **Delegating:** if the work splits into independent pieces or needs broad
  exploration, prompt the user to pick between inline and subagents — with
  your recommendation based on the size of the task — and read
  `references/multiagent.md` before dispatching any.

### 3. Debug

Any bug, test failure, or "this should be working" moment: stop and read
`references/debugging.md` **before proposing a fix**. No fixes before a
diagnosis — guess-and-check debugging costs more than it looks like it does,
and in some domains each guess is expensive to test.

### 4. Verify before claiming done

"Done" is a claim about evidence, not intention. Read
`references/verification.md` before claiming any task is complete. Minimum,
always: the project's linter/formatter is clean and relevant tests ran, with
real output shown.

### 5. Review

Before presenting work of any size, re-read the full diff with fresh eyes —
a small diff takes seconds to re-read, so this rung is never skipped. The
general method is in `references/verification.md`; the project-type skill
supplies the domain-specific checklist of silent killers to hunt for.

### 6. Git

Read `references/git.md` when branching, committing, or preparing to share
work.

## Reference files

| File | Read it when |
|---|---|
| `references/interview.md` | Anything about the task is unclear — any size, before asking |
| `references/planning.md` | Task is substantial — before writing the plan file |
| `references/testing.md` | Before writing any test |
| `references/multiagent.md` | Work could split across subagents — before dispatching any |
| `references/debugging.md` | Anything behaves unexpectedly — before proposing fixes |
| `references/verification.md` | Before claiming any task is done; before presenting work |
| `references/git.md` | Branching, committing, or sharing work |
