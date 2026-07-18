# README — LLM training projects

The README is the repo's only *committed* documentation — `docs/` and
`tmp/` are local-only, so the README is what a fresh clone (or a fresh
agent session) has. Its audience: future-you or an agent starting cold,
needing to run something within two minutes of reading.

**Never reference `docs/` or `tmp/` from the README** — they don't exist
in a fresh clone.

## Structure

```markdown
# <project>
One paragraph: what is being trained/investigated, and why.

## Setup
uv sync, required env vars (paths/tokens — names and what they point to),
and which commands run where, if the project spans machines.

## Usage
The real commands, copy-pasteable:
- smoke check:   uv run scripts/train.py --training smoke --model tiny
- real training: uv run scripts/train.py --training <t> --model <m> --seed <s>
- inference:     uv run scripts/infer.py ...
- export:        uv run scripts/export.py ...

## Variations
What each variation is *for* — one line per name, both axes:

**Model**
- tiny    — smoke runs and shape checks, never real results
- medium  — the main experiment scale
- large   — ...

**Training**
- smoke     — 50-step local sanity check
- full      — the standard recipe
- warmup2k  — tests whether longer warmup removes the early loss spike

Purpose only — the numbers live in variations.py. A variation whose
purpose you can't state in one line probably shouldn't exist. New
variation = new line here, same change.

## Layout
One line each for records/, checkpoints/, and any other output dirs —
what lands where, keyed by <model>/<training>/<seed>.

## Results (optional)
Current best runs: variation + seed + headline number. Update when a
better run replaces one.
```

## Rules

- **Commands must be real.** Every command in the README gets run before
  it's written down (verification evidence rules apply).
- **Don't duplicate what code documents.** The Variations section carries
  *purpose* only; the numbers live in `variations.py` — hyperparameter
  tables in the README drift silently. Point, don't copy.
- **Update in the same change.** If a change alters setup, commands, or
  layout, the README edit rides in the same commit — a stale README is
  worse than a sparse one.
- No training theory, no experiment history (that's what records are for).
