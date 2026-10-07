---
name: dev-workflow
description: The core development flow. Use at the start of any development task in any language or project type. Use together with the matching project-type skill (e.g. python-llm-training, rust-cli) when one exists, and invoke the companion skill for each phase as it arrives.
---

# Dev Workflow

The general workflow that drives the process. The user is familiar with this workflow, so you can communicate and work more efficiently. This skill covers the what and the when; the how lives in their own skills — invoke them when needed.

This skill is general guidance, not law: when the user's prompt says otherwise, the prompt wins — follow it without asking the skill's permission.

## How this layers

- **Companion skills** (interviewing, context-folder, planning, testing, debugging, verification, git-habits, subagent) define *how* and *when* to do each part of the process.
- **Project-type skills** (e.g. `python-llm-training`, `rust-cli`) define *domain content*: what questions matter, what verification proves, which tools to run.

## The flow

### 1. Plan

The general planning flow follows sizing → interviewing → plan → sign-off.

- **sizing**: there are no fixed sizing categories or metrics. Sizing is the process of understanding the project scope and applying that understanding to the rest of the workflow. When the user later adds or revokes a scoping decision, adjust the workflow dynamically.
- **interviewing**: every task gets an interview at the start when anything is unclear (see **interviewing** skill for the definition of clear and how to interview).
- **plan**: depending on the size of the project, planning could be a short paragraph of what is going to change or a multi-step iterative process (see **planning** skill).
- **sign-off**: requests explicit approval from the user before doing any editing; an update to the plan is not an approval. Ask the user to decide whether to use subagents if you think the task could benefit from them (see the **subagent** skill).

Project-type skills may define additional task types with their own process.

### 2. Implement

General rules only — domain rules live in the project-type skill:

- Match the existing codebase's conventions, style, and structure. Don't reorganize a repo without user permission.
- Always use the proper package manager to add dependencies (uv for Python, cargo for Rust) if one exists. If a language requires adding dependencies manually, like Java, find the up-to-date or desired version and information before editing.
- **Testing:** invoke the **testing** skill before writing any test.

### 3. Git Habits

Invoke the **git-habits** skill when branching, committing, or preparing to share work. A commit means the change has been approved by you (the orchestrator).

### 4. Debug

For any bug, test failure, or "this should be working" moment, act based on the complexity of the bug:

- Fix it and present it later in the summary when the bug is small and isolated
    - example: shape mismatch in the attention module
- Ask for user approval, with potential causes and proposed solutions, when the bug touches another part of the project or the computer
    - example: Python package corruption because of iCloud syncing

### 5. Verify before claiming done

"Done" is a claim about evidence, not intention. Invoke the **verification** skill before calling complete. Always use the project's linter/formatter and run relevant tests.

### 6. Review

Write a summary in your response, presenting the changes to the user directly. When the user asks for an update, start again from 1. Plan and repeat. If the user's updated task is small and clear, skip 1 and start from 2.

## Companion skills

| Skill | Invoke it when |
|---|---|
| `interviewing` | Anything about the task is unclear — any size, before asking |
| `context-folder` | Before creating or writing anything under `context/` |
| `planning` | When the task requires a proper planning phase with plan files |
| `test-writing` | Before writing any test or scratch check script |
| `subagents` | Work could split across subagents — before dispatching any |
| `verification` | Before claiming any task is done; before presenting work |
| `git-habits` | Branching, committing or pushing |
| `throwaway-html` | Creating a single-use HTML presentation |
| `homework-cli` | homework cli tool for spawning subagent of another family |
