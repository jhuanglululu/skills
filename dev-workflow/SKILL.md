---
name: dev-workflow
description: The core development FLOW. Use at the start of any development task in any language or project type. Always use together with the matching project-type skill (e.g. python-llm-training, rust-cli) when one exists, and invoke the companion skill for each phase as it arrives.
---

# Dev Workflow

The general workflow that drives the process. User is familiar with this workflow, so you can communicate and work more effeciently. This skill includes what and when. The how lives in their own skills, invoke them when needed.

This skill is general guidance, not law: when the user's prompt says otherwise, the prompt wins — follow it without asking the skill's permission.

## How this layers

- **Companion skills** (qusetion-asking, planning, testing, debugging, verification, git, subagent) define *how* and *when* to do each part of the process.
- **Project-type skills** (e.g. `python-llm-training`, `rust-cli`) define *domain content*: what questions matter, what risks to look for, what verification proves, which tools to run. 

## The flow

### 1. Size the task

Every task, regardless of size, gets an question-asking (when anything is unclear), sign-off before code, and review after — sizing changes how you work, but not the steps. How to ask is the **question-asking** skill.

- **Small** (a few minutes of work): clarify what's unclear, state your intended change in a sentence or two, and get a quick go-ahead, then code. No plan file, and no new tests. question-asking, sign-off, verification, and review is still required.
- **Substantial** (more than 30 minutes of work): question asking → plan file → sign-off → code → verification → review. With each state using the corresponding skill. Prompt the user to pick between working inline vs using subagents with your recommendation. Read the **subagent** skill for what and when to use subagents.

Project-type skills may define additional task types with their own process.

### 2. Implement

General rules only — domain rules live in the project-type skill:

- Match the existing codebase's conventions, style, and structure. Don't reorganize a repo without user permission.
- Always use proper package manager for add dependencies (uv for python, cargo for rust) if one exists. If a language requires manually adding dependecies, like Java, find up to date/desired version/information before editing.
- **Testing:** invoke the **testing** skill before writing any test.

### 3. Debug

Any bug, test failure, or "this should be working" moment: stop and invoke the **debugging** skill **before proposing a fix**. No fixes before a diagnosis.

### 4. Verify before claiming done

"Done" is a claim about evidence, not intention. Invoke the **verification** skill before calling complete. Always use the project's linter/formatter and run relevant tests.

### 5. Review

Before presenting work of any size, check what files changed, report any unexpected file change. Use the **verification** skill and any relevant per-project verification.

### 6. Git

Invoke the **git** skill when branching, committing, or preparing to share work. A commit means approved by you(the main model).

## Companion skills

| Skill | Invoke it when |
|---|---|
| `interview` | Anything about the task is unclear — any size, before asking |
| `planning` | Task is substantial — before writing the plan file |
| `testing` | Before writing any test or scratch check script |
| `subagent` | Work could split across subagents — before dispatching any |
| `debugging` | Anything behaves unexpectedly — before proposing fixes |
| `verification` | Before claiming any task is done; before presenting work |
| `git` | Branching, committing, or sharing work |
