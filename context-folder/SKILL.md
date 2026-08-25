---
name: context-folder
description: What the project's context/ folder is, its layout, and the rules every file in it follows. Use before creating or writing anything under context/ (plans, designs, presentations, subagent homework), when setting up a new project, or whenever the user mentions the context folder.
---

# The context/ Folder

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

`context/` is the project's local-only working memory. It belongs to the user and the agents working on the project — **never committed**, never referenced from committed docs (README, code comments). Committed documentation must stand on its own without it.

## Layout

```
context/
├── .gitignore       # contains "*" — nothing in context/ is tracked
├── presentations/   # throwaway HTML shown to the user (see throwaway-html skill)
├── plans/           # plans saved for other sessions (see planning skill)
├── designs/         # things that were decided (see planning skill)
├── preferences/     # user preferences or working styles
└── subagents/       # homework files for other-family subagents (see homework-cli skill)
```

Create `context/` and its `.gitignore` together on first use — never one without the other. The folder-local `*` ignore keeps the root `.gitignore` free of surprises (see the **git-habits** skill for the two-tier gitignore rule). Only create subfolders as they are needed.

## Format rule: pick by audience

- **User-facing → single-file HTML** (`presentations/`): explain with *examples* instead of words and sentences — that's the whole reason for HTML over markdown. These files are **throwaway**: never reference them from plans, designs, or subagent prompts.
- **Agent-facing → markdown** (`plans/`, `designs/`, `preferences/`, `subagents/`): explain clearly with words so an agent without this conversation's context can understand easily. This serves both subagents and future sessions without context.

## Naming

Files in `presentations/` and `plans/` are named `<id>-<slug>`. `<id>` is the next sequential number within that folder, so the user can say "plan 3" or find the newest presentation easily. Timestamp goes on a line under the title for markdown files.

## Referencing

- Reference plans and designs by filename (`context/plans/3-foo.md`) — from other context files, from subagent prompts, in conversation.
- Never reference `presentations/` from anything durable.
- Never reference `context/` from committed files.

## Lifecycle

- **Resuming work**: the newest plan is the first thing to read when work continues in a later session.
- **Updating**: plans and designs are rewritten to read as current; presentations are never updated — write a new one when the topic changes.
- **Cleanup**: `context/` is disposable by design. Never delete the user's files in it without asking.
