# Implementation — LLM training specifics

General implementation rules are in dev-workflow (conventions, manifests)
and the testing skill (risk-based principle). This file defines the
training-specific conventions. The value behind most of them is
**reproducibility**: any result must be re-creatable from the code, a model
variation, a training variation, and a seed — nothing else.

## Project layout

Typical shape (adapt to what an existing repo already uses):

```
project/
├── pyproject.toml
├── scripts/          # thin entry points: train.py, eval.py
├── src/<pkg>/        # data.py, model.py, variations.py, ...
├── tests/
├── datasets/
├── records/          # records/<model>/<training>/<seed>/record.jsonl (root-gitignored)
├── checkpoints/      # checkpoints/<model>/<training>/<seed>/... (root-gitignored)
├── tmp/              # scratch check scripts — local-only, own "*" .gitignore
└── docs/             # plans & designs — local-only, own "*" .gitignore
```

`records/` and `checkpoints/` share one addressing scheme —
`<root>/<model>/<training>/<seed>/` — and so does any other per-run file
(metadata, eval dumps, plots). Everything a run produces is findable from
its model variation, training variation, and seed.

## Default stack

Beyond torch, every project gets these by default (`uv add`):

- **tokenizers** — a tokenizer JSON file is provided in most cases. Load it
  with the `tokenizers` library, and **inspect it for special tokens**
  (BOS/EOS/PAD/chat markers) instead of assuming ids or names — wrong
  special-token assumptions are a classic silent data bug.
- **safetensors** — all weights on disk (see checkpoints section).
- **tqdm** — training progress display (see progress section).
- **numpy** — hide torch's warning message.

## Variations, not config sprawl

Configuration is hardcoded as named variations in code with pydantic (e.g. a registry in
`variations.py`), on **two independent axes**:

- **Model variations** — architecture: dims, layers, heads, vocab —
  e.g. `tiny`, `medium`, `large`.
- **Training variations** — the recipe: steps, batch size, LR schedule, data
  amount — e.g. `smoke`, `full`, `warmup2k`.

Any model can be paired with any training recipe:

```
uv run scripts/train.py --training smoke --model large
uv run scripts/train.py --training full  --model medium --seed 42
```

Never expose architecture or recipe through free-form CLI args or env vars —
a variation that lives in code is documented, diffable, and reviewable; a
run assembled from ad-hoc CLI overrides is unreproducible the moment the
shell history is gone.

- **CLI args (training scripts):** `--model`, `--training`, `--seed`.
  Nothing else. (Inference scripts have their own small arg set — see the
  inference section.)
- **Env vars:** paths and tokens (data roots, `HF_TOKEN`, …). Nothing else.
- New experiment = new named variation in code, not a flag combination.

Every project includes a **`smoke` training variation** — ~32 samples,
**~50 steps** (a real epoch is thousands of steps; 50 is enough to watch
loss move and exercise logging/checkpointing) — and a **`tiny` model
variation**, so `--training smoke --model tiny` finishes on the development
machine in about two minutes. The verification ladder depends on both existing. Because
the axes are independent, `--training smoke --model large` also works as a
quick shape/memory check of the big model.

## Records

Every run appends to `records/<model>/<training>/<seed>/record.jsonl`.
Metadata line first:

```json
{"type": "meta", "model": "medium", "training": "warmup2k", "seed": 42, "config": {...}, "git_commit": "abc123", "baseline": "base", "started": "2026-07-14T17:30:00"}
```

Then one line per logging interval:

```json
{"type": "step", "step": 100, "loss": 2.431, "lr": 3e-4, "grad_norm": 0.82, "tokens_seen": 1638400, "sec_per_step": 0.41}
```

Rules: append-only, one flat JSON object per line (nesting beyond `config`
makes ad-hoc pandas/`json.loads` analysis annoying), always include `step`.
Log `grad_norm` — it's the earliest warning for most instabilities.
Evaluation lines use `"type": "eval"` in the same file. Any new mechanism
(new loss term, new schedule) must be visible in the record so its effect
can be inspected afterwards.

Seed everything before step 0: `random`, `numpy`, `torch` (dataloader
workers too) — from the CLI seed, so model + training + seed fully
determines the run.

## Progress display (tqdm)

The record.jsonl is for analysis; tqdm is what the user actually watches.
Default bar — variation, epoch, percentage, bar, step/total-per-epoch,
elapsed<remaining, sec/step, then loss and val:

```
medium/warmup2k e2  45%|████▌     | 1350/3000 [09:13<11:16, 0.41s/it, loss=2.431, val=2.512]
```

(`desc` = `<model>/<training> e<epoch>`; loss/val via `set_postfix`; the
step counter is per-epoch.)

On **every validation**, `tqdm.write` a persistent line — epoch, step,
elapsed time, train loss, val loss, and **diff = val − train**, the
generalization gap — so the val history stays readable above the moving
bar. These are mostly small models, where overfitting is the default
failure: a steadily growing diff is the earliest visible warning, so it
earns a permanent column:

```
e2 | step  1350/3000 | 09:13 | loss 2.431 | val 2.512 | diff +0.081
e2 | step  1500/3000 | 10:15 | loss 2.398 | val 2.489 | diff +0.091
```

**Align the digits**: fixed-width fields and fixed decimal places
(`step {step:>5}`, `loss {loss:6.3f}`, `diff {diff:+6.3f}`, zero-padded
times) so consecutive lines form clean columns. A format that jitters
per-line is unreadable at a glance, which defeats the point.

## Checkpoints

- Path: `checkpoints/<model>/<training>/<seed>/<step>.safetensors`, with
  `current.safetensors` for the latest state used by resume.
- **safetensors, never `.pt`** — safe to load, mmap-able, portable.
  Non-tensor state (step counters, RNG states, config snapshot) goes in a
  sidecar JSON with the same stem (`<step>.json`).
- **Resume-from-checkpoint is a default feature** of every training script —
  build it in unless the user explicitly opts out. Remote runs die; a script
  that can't resume gambles the whole run on uptime.
- **Keep the 3 best checkpoints by default** (by the run's headline metric),
  plus `current`; prune the rest as training proceeds.
- **Provide an export script** (e.g. `scripts/export.py`) that strips a
  training checkpoint down to `model.safetensors` — model weights only, no
  optimizer/scheduler/RNG state — written next to the source checkpoint.
  Training state is most of a checkpoint's size; exporting first makes
  pulling a model off the training machine much faster, and the
  weights-only file is what inference (torch or MLX) loads anyway.

## Device discipline

Which devices matter depends on the project's compute setup (per the
starting question in the planning reference), so never assume — code must
run on `cuda`, `mps`, and `cpu`:

- One `get_device()` helper; never a literal `.cuda()` or `"cuda:0"` in
  model/data code.
- **On cuda, pick a free GPU via `nvidia-smi`** (memory ≈ empty, no compute
  processes) before binding — training boxes are often shared, and landing
  on an occupied GPU slows both jobs or OOMs one of them. Put the check in
  `get_device()` so every script gets it.
- Guard non-portable features (`torch.compile`, bf16, flash attention)
  behind the variation definitions with safe defaults, so
  `--training smoke --model tiny` runs on MPS/CPU.
- **If `torch.compile` is on, recompilation during training must not
  happen** — steady-state recompiles silently destroy throughput. The usual
  trigger is varying shapes (unpadded batches, a changing sequence length):
  pad to fixed shapes or mark dims dynamic. Verifying zero steady-state
  recompiles is part of the smoke run (see verification reference); when
  steps are mysteriously slow, check recompiles first (see debugging
  reference).

## Backend split: torch vs MLX

- **torch (cuda/mps/cpu): training *and* inference.** The torch inference
  path is what training-time evals and remote generation use.
- **MLX: inference only** (when the project includes Apple-silicon
  inference), never the training path. These are experiments with custom
  architectures, so `mlx_lm` generally won't load
  them — **write a custom MLX inference module** (e.g.
  `src/<pkg>/mlx_infer.py`): mirror the torch model definition in MLX and
  load the same safetensors weights directly. Keep the import boundary
  clean — torch modules never import mlx and vice versa — and when MLX
  output looks wrong, compare it against torch inference from the same
  checkpoint before blaming the checkpoint.

## Inference scripts

One CLI shape for inference entry points (torch or MLX):

- **Model:** checkpoint path or `--model` variation name.
- **Sampling:** `--top-k`, `--top-p`, `--rep-pen`, `--seed` — every one
  with a sensible default, so a bare invocation just generates.
- **Print decode speed** (tokens/sec) after generating. Skip prefill
  measurement — it's hard to measure honestly in a simple script, and a
  misleading number is worse than none.

## Shape and data discipline

Silent shape bugs are the signature failure of training code — broadcasting
makes wrong shapes run without crashing.

- Assert shapes at function boundaries where a mistake would propagate
  silently: `assert logits.shape[:2] == labels.shape`, with a comment naming
  the expected shape `(B, T, V)`.
- Comment expected shapes on non-obvious tensor ops.
- The single most bug-prone spot in LLM training is **label/input alignment
  and masking** (shift-by-one for causal LM, prompt masking with `-100` for
  SFT, padding masks). Write this code once, in a pure function, with unit
  tests — never inline it in the loop.

## Risk-based testing, applied to training code

**Unit tests required** — tokenization/formatting of examples, label masking
and shifting, batching/collation/padding, data filtering and dedup, LR
schedules, metric computation. Cheap to test (CPU, milliseconds) and exactly
where silent quality bugs live. The highest-value test pattern: build one
tiny hand-written example and assert the *exact* expected token IDs and
label values, `-100`s included.

**Smoke-tested, not unit-tested** — the training loop itself, checkpoint
save/resume, logging wiring. Their correctness is only visible end-to-end;
that's what `--training smoke --model tiny` is for.

Run tests with `uv run pytest`. Tests must not download models or data —
build tiny fixtures (a 100-token tokenizer, a 4-layer model) in the test.
