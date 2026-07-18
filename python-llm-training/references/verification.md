# Verification — LLM training specifics

Evidence rules and the ladder concept (climb as far as the change warrants,
paste real output, state skipped rungs) are defined in the verification
skill. This file defines the concrete rungs for training projects.

## The rungs

**1. Lint, format & type check — every change.**
```
uv run ruff format . && uv run ruff check . && uv run pyrefly check
```
(If the project uses different tooling, use that; if it has none, add ruff
and pyrefly to dev dependencies.)

**2. Unit tests — every change touching testable logic.**
`uv run pytest`, full suite. The risk-based rule from the implementation
reference is enforced here: changed masking/tokenization/data logic with no
covering test means writing the test now.

**3. Local smoke run — every change touching the training path.**
Run `--training smoke --model tiny` end-to-end on the development machine
(~50 steps, < 2 min). It must:

- complete without error,
- produce a JSONL with a meta line and step lines,
- show loss actually decreasing over the smoke run,
- start near `ln(vocab_size)` for a fresh causal LM (see debugging
  reference),
- if `torch.compile` is enabled: **zero recompilations after the first
  steps** (run with `TORCH_LOGS=recompiles` or check dynamo counters) —
  steady-state recompiles silently destroy training throughput.

If the change touches checkpointing: save mid-smoke, resume, confirm loss
continues rather than resetting.

**4. Real execution — when the deliverable is behavior, not training.**
Inference/eval scripts, data pipeline CLIs, plotting tools: run them on real
(small) input and look at the output. For MLX inference work, generate from
a real prompt and read the generation; compare against torch output from the
same checkpoint when correctness is in question.

**5. Launch readiness — before any real training run.**
The expensive rung. Before handing the user a launch command:

- smoke run passed on the *same commit* being launched,
- config reviewed against the run's stated plan (hypothesis, metric,
  decision criterion exist — see planning reference),
- checkpointing + resume verified (rung 3), because long runs die (SSH
  drops, preemption, shared-box surprises),
- model and training variations defined in code (not flag-assembled), seed
  explicit, baseline set — so the run's
  `records/<model>/<training>/<seed>/record.jsonl` metadata is right,
- cost estimate stated (steps × sec/step from a comparable run's record).

Launching a real training run is the user's call. Present the command;
don't run it unasked.

Some projects genuinely can't climb every rung (e.g. no runnable smoke path
yet) — per the verification skill, say which rungs were skipped and why.

## Silent killers — the training self-review checklist

The verification skill's self-review says to hunt the domain's silent
killers in the full diff before presenting. For training code, they are:

- **Alignment:** labels shifted correctly? prompt tokens masked with `-100`?
  padding excluded from loss?
- **Off-by-one:** sequence slicing, warmup boundaries, resume step counting.
- **Leakage:** eval data reachable from the training set? metric computed on
  tokens the model was trained on?
- **Randomness:** any new randomness that isn't seeded (augmentation,
  sampling, dataloader workers)?
- **Portability:** hardcoded device, absolute path, or personal directory
  that breaks on another machine?
- **Config honesty:** every new behavior reachable from config, defaults
  matching previous behavior (a changed default silently changes everyone's
  runs)?
- **Log completeness:** new mechanisms visible in the JSONL (per the
  implementation reference) so their effect can be inspected afterwards?
