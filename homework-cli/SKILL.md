---
name: homework-cli
description: Usage and guides for using `homework` cli tool that spawns subagents for other model family (gpt).
---

# Homework Cli

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

Use `subagent` skill too if not loaded. Source code for `homework` can be found at /Users/jhuanglululu/Documents/TypeScript/homework.

## The homework file

One markdown file per task. Frontmatter is the complete config (there is no
global config file); the body is the task prompt.

Run `homework providers list` to see registered endpoints before writing; pi's built-in providers (`openai`, `openrouter`, ...) are referenced by name.

```markdown
---
provider: cliproxy          # required; a pi provider or an endpoint from `homework providers add`
model: gpt-5.6-sol          # required; model id as the provider knows it, may contain slashes
tools: [read, grep, ls, fetch, search]
write_dirs: [./src]         # optional; write/edit rejected outside these dirs
read_dirs: []               # optional; empty/absent = unrestricted reads
cwd: .                      # optional, relative to this file (default: file's dir)
max_turns: 40               # optional, default 50
thinking: low               # optional: off|minimal|low|medium|high; some models (OpenRouter muse-spark) reject off
---
Find all TODO comments under src/ and summarize them by module.
```

Both `provider` and `model` are required — there is no combined `provider/model` form. Models missing from pi's catalog still work (e.g. `provider: openrouter` / `model: meta/muse-spark-1.2-contributor`); they are synthesized from the provider's endpoint and assumed to support reasoning and images. Models in pi's catalog use the catalog's capabilities, so a catalog model marked text-only silently drops images from `read`.

Restriction is structural: tools not listed are never registered on the pi session, and file tools enforce the path scoping before executing. `bash` bypasses path scoping by nature, so grant it deliberately.

Put the homework file at `context/subagents/{slug}/{slug}.md` (see the **context-folder** skill) so that future sessions can reference it.

## Tools

Built-ins from pi: `read`, `bash`, `edit`, `write`, `grep`, `find`, `ls`. Web tools: `fetch` (Firecrawl) and `search` (Tavily).

## Results

For `task.md` the server writes siblings, overwritten on re-run:

- `task.result.md` — final assistant message + status footer. What you read.
- `task.transcript.jsonl` — every message, tool call/result, and bash review decision, streamed live while the task runs. Parse with python if you need to extract information.

## Follow-up, fork, and steering

Continue a **finished** task by writing a new homework file with `continue:`; omitted fields inherit the parent's effective config, any field can be overridden (e.g. switch models mid-conversation):

```markdown
---
continue: ./greet.md
---
Now also translate hello.txt to French.
```

History comes from the parent's sibling transcript and the child's transcript starts with the imported history. Fork = two files continuing the same parent; chains read only their direct parent. Continuing a still-running parent is rejected.

Steer a **running** task with `homework send <id|file> "message"` — the message is delivered at the next turn boundary and the task keeps going in the same transcript. Sending to a finished task errors.

## CLI

```
homework run <file.md> [--wait]   submit; --wait blocks silently, prints the final message, exits 0 (completed) / 1 (error, killed, max_turns) / 2 (bad file or usage)
homework status <id|file>         state + last message + error
homework list                     all tasks
homework kill <id|file>           cancel a running task
homework send <id|file> <msg>     steer a running task (next turn boundary)
homework providers add <endpoint> --url <url> --api <openai|anthropic> [--token <key> | --token-env <VAR>]
                                  register an endpoint; any model id works under it, no server restart needed
homework providers list | remove <endpoint>
```

Without `--wait`, `run` prints the task id and returns; poll with `status`.
