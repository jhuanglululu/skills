# README for LLM training projects

The README is the repo's only *committed* documentation. `context/` and `tmp/` are local-only, so the README is all that a fresh clone (or a fresh agent session) has. Its audience is a future reader or an agent starting cold, who needs to run something within two minutes of reading.

**Never reference `context/` or `tmp/` from the README.** They don't exist in a fresh clone.

## Structure

```markdown
# PROJECT NAME

One paragraph: what is being trained or investigated, and why.

## Results (optional)
Current best runs: variation, seed, and headline number. Update when a better
run replaces one. Charts are encouraged. Chart scale must not be misleading:
if a result increases from 105 to 110, don't put the baseline at 100 and make
it look like a 2x increase.

## Setup
uv sync, required env vars (paths and tokens: their names and what they point
to), and which commands run where, if the project spans machines.

## Usage
The real commands, copy-pasteable:
- smoke check:   uv run scripts/train.py --training smoke --model tiny
- real training: uv run scripts/train.py --training <t> --model <m> --seed <s>
- inference:     uv run scripts/infer.py ...
- export:        uv run scripts/export.py ...

## Variations
What each variation is *for*, one line per name, on both axes:

**Model**
- tiny    — smoke runs and shape checks, never real results
- medium  — the main experiment scale
- large   — ...

**Training**
- smoke     — 50-step local sanity check
- full      — the standard recipe
- warmup2k  — tests whether longer warmup removes the early loss spike

Purpose only. The numbers live in variations.py. A variation whose purpose
you can't state in one line probably shouldn't exist. A new variation gets a
new line here, in the same change.
```

## Rules

- **Commands must be real.** Run every command in the README before writing it down.
- **Don't duplicate what code documents.** The Variations section carries *purpose* only. The numbers live in `variations.py`. Hyperparameter tables in the README drift silently. Point to the code instead of copying it.
- **Update in the same change.** If a change alters setup, commands, or layout, the README edit goes in the same commit. A stale README is worse than a sparse one.
- **No training theory and no experiment history.** Records exist for history.
