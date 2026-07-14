# Planning — LLM training specifics

Process (sizing, how to ask, plan file format) is defined in dev-workflow.
This file covers what's specific to training projects.

## Planning a training run

A training run meant to answer a question gets sized like any other task —
some are small (a quick LR sanity check), some substantial (a multi-day
comparison); judge each one by dev-workflow's usual cost-of-a-mistake rule.
What's training-specific is *what the plan covers*: a run without a decision
criterion is GPU time spent to produce a shrug. Whatever the size — in the
plan file if substantial, directly in conversation if small — write down
before launching:

- **Hypothesis.** "Increasing LR warmup to 2k steps will remove the early
  loss spike." Not "try some warmup values."
- **Metric + comparison.** What number, measured how, compared against which
  baseline run's JSONL?
- **Decision criterion.** What result makes you adopt the change, and what
  makes you discard it? Decide *before* seeing the data — post-hoc criteria
  drift toward whatever the run produced.
- **Cost estimate.** Steps × time-per-step from a previous run's JSONL. If a
  smoke run can answer the question at 1% of the cost, run that first.

A run is identified by **model variation + training variation + seed**
(`records/<model>/<training>/<seed>/` — see the implementation reference).
Name a new variation after what changed relative to its baseline, e.g.
training variation `warmup2k` vs `base`; the baseline goes in the run's
metadata line.

## Questions that matter for training work

When interviewing for substantial training tasks (per dev-workflow: read the
repo first, batch the questions), these are the domain questions worth
asking if the repo doesn't answer them:

1. **Scale facts.** Model size, dataset size, sequence length, expected
   runtime. A design that's right at 100M params is often wrong at 7B.
2. **Where does it run?** Local-only (MLX/MPS constraints apply) or remote
   (must survive SSH disconnects, must checkpoint)?
3. **What does done look like?** A metric, a working script, a plot the user
   can look at?

## What belongs in a training plan's Risks section

Training risks are silent by default — list them with the check that catches
each one. The usual suspects: label/mask misalignment, train/eval leakage,
tokenizer/template mismatch between training and inference, instability at
scale (fine in smoke, NaN at full LR/batch), dataloader bottlenecks, and
checkpoint/resume gaps for long remote runs.
