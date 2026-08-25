---
name: interviewing
description: When and how to ask user questions using the AskUserQuestion tool. Use before any planning and implementation to align you and the user.
---

# Interviewing

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## What the interview is for

The interview exists to **align your mental model with what the user has in mind** — direction, intent, priorities. It is *not* for pinning down every design decision. Details are your job: make good decisions, then **propose them** (in the intent statement or plan file) where the user can redirect cheaply.

When designing public interface (public api, user facing ui/tool), interview the user deeply until every design branch is visited and user confirms you have reached a shared understanding. A good way to verify is to write a short proposal in `context/designs` for the interface shape.

The interview is also for **checking constraints** — facts about the real setup that live outside the repo and can't be inferred. Design decisions are yours to propose; constraints are facts to collect. Per-project skills have lists of questions that you can ask.

Applies to **every task size**. Small tasks skip the plan file, not the alignment.

## When and how to ask

- **Read before asking.** Explore the repo, existing docs, git logs, and plan files first.
- **Ask what you cannot infer** from existing information — what lives only in the user's head. If it's a detail you could make a good call on, make the call and state it instead of asking.
- **Ask about unclear or conflicting intent.** When the user proposes something whose intent you can't reasonably guess, ask instead of assuming it is correct. The user can be wrong.
- **Batch questions** instead of one at a time, with your recommended answer stated where you have one. When they don't all fit, group related ones together so each round covers a coherent topic. Ask more questions if something is still unclear — aim for alignment instead of completing fast.
- **Give examples** when the answer affects the final shape of the work — e.g. choosing between libraries, API design, visual layout. Don't put examples in option or tool call descriptions; present the examples as text before invoking the AskUserQuestion tool for better formatting. When the examples are long, write them to `context/presentations/` (see the **throwaway-html** and **context-folder** skills).

When the user answers with **you decide**, tell them your pick in the next response if you have enough information, or after asking more questions if you don't.

When asking through the AskUserQuestion tool, **always use multi-select**.

- The user's real answer often combines options or adds their own, and single-select forces a false choice.
- The notes field only appears in multi-select.
- When a user selects multiple options where you expected one, this signals misalignment; ask follow-up questions to resolve it.

## Answers are a snapshot, not ground truth

Answers are guidance for the current moment. The user may — and often will — change their mind partway into the project, and that's a normal part of the process, not a violation of the plan. So:

- When a change of direction lands, update the plan file so the record matches reality (see the **planning** skill).
- If two answers or ideas conflict, ask the user explicitly to resolve it instead of making assumptions, and bring up any potential pros and cons.
