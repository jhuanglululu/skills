# Interviewing

## What the interview is for

The interview exists to **align your mental model with what the user has in
mind** — direction, intent, priorities. It is *not* for pinning down every
design decision. Details are your job: make good decisions, then **propose
them** (in the intent statement or plan file) where the user can redirect
cheaply. An interview that asks the user to specify everything just exports
your work back to them; an interview that asks nothing risks building the
wrong thing confidently. Aim between: ask enough to be pointed the right
way, decide the rest yourself, visibly.

Applies to **every task size**. Small tasks skip the plan file, not the
alignment — a wrong assumption on a one-line change still produces the wrong
change. What scales with size is depth: a small task might need one
clarifying question folded into the intent statement; a substantial task
gets a proper interview before the plan is written.

## When and how to ask

Questions are cheap for you and expensive for the user — each round-trip
interrupts them. So:

- **Read before asking.** Explore the repo, existing docs, and plan files
  first. Never ask something the codebase already answers.
- **Ask only what you cannot decide-and-propose yourself**: intent,
  priorities, constraints that live in the user's head. If it's a detail you
  could make a good call on, make the call and state it instead of asking.
- **Batch questions** into one round, most important first, with your
  recommended answer stated where you have one. Aim for one round of
  questions, not a drip.
- **State assumptions instead of asking about trivia.** "I'm assuming X; say
  so if that's wrong" keeps momentum while staying correctable.

When asking through a structured question tool (e.g. AskUserQuestion):
**always use multi-select** — the user's real answer often combines options
or adds their own on top, and single-select forces a false choice.

## Longer examples: `docs/examples/`

When a question or proposal needs more example than fits in the tool call,
write a **single-file HTML** page to `docs/examples/` and point the user at
it. These are user-facing, so per the planning reference's format rule:
explain with examples, not prose. Specifics:

- One file per tool call; if the call has multiple questions, use tabs to
  navigate between them.
- Code shown with syntax highlighting (a CDN library is fine — assume an
  internet connection).
- Keep it simple; reference plans or designs by filename if needed.
- These files are **throwaway**: never reference them from plans or designs,
  and don't maintain them after the question is answered.

## Answers are a snapshot, not ground truth

Interview answers are guidance for the current moment. The user may — and
often will — change their mind partway into the project, and that's a normal
part of the process, not a violation of the plan. So:

- Recent direction always beats an earlier answer. Don't argue from "but you
  said earlier" — just confirm you've caught the change and follow it.
- When a change of direction lands, update the plan file so the record
  matches reality (per the planning reference).
- If new evidence makes an earlier answer look wrong, don't silently obey it
  or silently override it — surface what you found and let the user re-decide.

*What* to ask about is domain knowledge — the project-type skill lists the
questions that matter for its domain.
