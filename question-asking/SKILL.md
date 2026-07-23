---
name: question-asking
description: When and how to ask user questions using the AskUserQuestion tool.
---

# Interviewing

General guidance, not law: when the user's prompt says otherwise, the prompt wins.

## What the interview is for

The interview exists to **align your mental model with what the user has in mind** — direction, intent, priorities. It is *not* for pinning down every design decision. Details are your job: make good decisions, then **propose them** (in the intent statement or plan file) where the user can redirect cheaply.

The interview is also for **checking constraints** — facts about the real setup that live outside the repo and can't be inferred. Design decisions are yours to propose; constraints are facts to collect. Per-project skills have lists of questions that you should ask.

Applies to **every task size**. Small tasks skip the plan file, not the alignment.

## When and how to ask

- **Read before asking.** Explore the repo, existing docs, and plan files first. Never ask about facts you can find in the project.
- **Ask only what you cannot decide-and-propose yourself**: intent, priorities, constraints that live in the user's head. If it's a detail you could make a good call on, make the call and state it instead of asking.
- **Ask for unclear intention.** User propose something that you can't reasonly guess the intention. Ask instead of assuming it is correct. The user can be wrong. 
- **Batch questions** into one round, most important first, with your recommended answer stated where you have one. Aim for one round of questions.
- **State assumptions instead of asking about trivia.** "I'm assuming X; say so if that's wrong" keeps momentum while staying correctable.

When asking through a structured question tool (e.g. AskUserQuestion): **always use multi-select** — the user's real answer often combines options or adds their own on top, and single-select forces a false choice.

## Longer examples: `docs/examples/`

When a question or proposal needs more example than fits in the tool call, write a **single-file HTML** page to `docs/examples/` and point the user at it. These are user-facing, so follow the **planning** skill's format rule: explain with examples, not prose. Specifics:

- One file per tool call; if the call has multiple questions, use tabs to navigate between them.
- Code shown with syntax highlighting (a CDN library is fine — assume an internet connection).
- Keep it simple; reference plans or designs by filename if needed.
- These files are **throwaway**: never reference them from plans or designs, and don't maintain them after the question is answered.

## Answers are a snapshot, not ground truth

Answers are guidance for the current moment. The user may — and often will — change their mind partway into the project, and that's a normal part of the process, not a violation of the plan. So:

- When a change of direction lands, update the plan file so the record matches reality (see the **planning** skill).
- If two answer/idea conflict, ask user explicitly to resolve it instead of making assumptions and bring up any potential pros and cons.

*What* to ask about is domain knowledge — the project-type skill lists the questions that matter for its domain.
