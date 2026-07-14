---
name: python-llm-training
description: Domain-specific workflow for Python LLM training projects — PyTorch/HuggingFace training code, uv-managed environments, JSONL experiment logs, and a local-Mac (MLX/MPS) + remote-GPU split. Use this whenever the task touches model training, fine-tuning, dataset preparation, tokenization, training loops, loss/convergence debugging, evaluation harnesses, checkpoint handling, or local inference with MLX — from the first message of the task, before writing any code. Always use together with the dev-workflow skill, which owns the general process this skill plugs into.
---

# Python LLM Training

This skill is general guidance, not law: when the user's prompt says
otherwise, the prompt wins.

Domain knowledge for LLM training projects. **This skill extends
`dev-workflow`** — invoke that too if it isn't already loaded. Process rules
(task sizing, plan files, debugging discipline, evidence rules, git) live
there and are not repeated here; this skill supplies what's *different* about
training code.

What's different, in one sentence: **the expensive failure mode is silent.**
A bug in a data pipeline or loss mask doesn't crash — it burns GPU-hours
producing a subtly worse model. Everything here exists to catch problems at
the cheapest moment: at plan time, at code time, or in a 2-minute local smoke
run — never 6 hours into a remote training job.

## Ground rules

- **uv only.** `uv run`, `uv add`, `uv sync`. Never bare `pip install` or
  manually activated venvs. If a project lacks `pyproject.toml`, set it up
  with `uv init` before adding code.
- **Two-machine reality.** This Mac does development, testing, and MLX/MPS
  inference. Real training runs happen on a remote GPU box. Code must be
  device-agnostic, and every training script must be runnable at toy scale
  locally.
- **JSONL is the experiment record.** Every run appends to
  `records/<model>/<training>/<seed>/record.jsonl` — metadata line first,
  then per-step metrics. A run is fully identified by code + model
  variation + training variation + seed; reproducibility is the point.
  Schema in `references/implementation.md`.

## Domain deltas per phase

- **Planning:** training runs are sized like any other task (dev-workflow's
  small/substantial rule), but a run meant to answer a question always needs
  a hypothesis, metric, and decision criterion written down before launch —
  at a weight matching its size. That, what to ask, and what belongs in a
  training plan's Risks: `references/planning.md`.
- **Implementation:** hardcoded named variations on two axes — model
  (architecture) and training (recipe) — selected via `--model`/`--training`
  (never architecture through free-form flags/env), required `smoke`
  training and `tiny` model variations, records and checkpoint conventions
  (safetensors, resume by default, keep 3 best), device and shape/masking
  discipline, and which parts of training code get unit tests vs smoke
  runs: `references/implementation.md`.
- **Debugging:** what "minimum scale" means here, the two universal sanity
  checks (overfit-one-batch, loss ≈ ln(V) at step 0), and the standard
  failure catalog (NaN loss, flat loss, OOM, slow steps, bad-at-inference):
  `references/debugging.md`.
- **Verification:** the concrete rungs of the ladder for training code —
  ruff, pytest, the local smoke run, real inference runs, and remote launch
  readiness — plus the self-review checklist of training's silent killers:
  `references/verification.md`.

## Reference files

| File | Read it when |
|---|---|
| `references/planning.md` | Planning substantial training work, or any training run |
| `references/implementation.md` | Writing/modifying any project code |
| `references/debugging.md` | Anything behaves unexpectedly — before proposing fixes |
| `references/verification.md` | Before claiming any task is done |
| `references/readme.md` | Creating or updating the project README |
