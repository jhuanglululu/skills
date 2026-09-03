# Training

## Variations, not config sprawl

Configuration is hardcoded as named variations in code, using pydantic (for example, a registry in `variations.py`), on **two independent axes**:

- **Model variations**: the architecture. Dims, layers, heads, vocab. For example `tiny`, `medium`, `large`.
- **Training variations**: the recipe. Steps, batch size, LR schedule, data amount. For example `smoke`, `full`, `warmup2k`.

A training recipe can be restricted to specific models. This is useful when comparing model architectures.

Example CLI usage:

```bash
uv run scripts/train.py -m large -t smoke -s 0
uv run scripts/train.py --model medium --training full --seed 42
```

Never expose architecture or recipe through free-form CLI args or env vars. A variation that lives in code is documented, diffable, and reviewable. A run assembled from ad-hoc CLI overrides cannot be reproduced once the shell history is gone.

- **CLI args (training scripts):** `--model`, `--training`, `--seed`.
    - Seed everything before step 0: `random`, `numpy`, and `torch`, including dataloader workers, so that model, training, and seed fully determine the run.
- **Env vars:** tokens such as `HF_TOKEN`. Nothing else.
- A new experiment is a new named variation in code, not a flag combination.
- Temporary variations are allowed for debugging.

Every project should include a `smoke` training variation and a `tiny` model variation for smoke testing. This is especially helpful for checking the forward and backward pass.

## Records

Every run appends to `records/<model>-<training>/<seed>-record.jsonl`. Records are append-only, one flat JSON object per line.

The first line is metadata and includes:

- `"type": "meta"`
- model and training variation
- seed
- commit hash of the source code
- start time
- any extra metadata

Example:

```json
{"type": "meta", "model": "medium", "training": "warmup2k", "seed": 42, "git_commit": "abcd123", "started": "2026-07-14T17:30:00"}
```

Then one line per logging interval, which includes:

- `"type": "step"`
- step, loss, lr, grad_norm, timestamp, sec_per_step
- any other information

```json
{"type": "step", "step": 100, "loss": 2.431, "lr": 3e-4, "grad_norm": 0.82, "timestamp": "2026-07-14T17:30:00", "sec_per_step": 0.41, "tokens_seen": 1638400}
```

Evaluation lines use `"type": "eval"` in the same file.

## Progress display (tqdm)

The record file is for analysis. The tqdm bar is what the user watches. The default bar includes:

1. variation
2. epoch
3. percentage
4. bar
5. step/total-per-epoch
6. elapsed<remaining
7. sec/step
8. loss
9. val

Example:

```
medium/warmup2k e2  45%|████▌     | 1350/3000 [09:13<11:16, 0.41s/it, loss=2.431, val=2.512]
```

Notes:

- `desc` is `<model>/<training> e<epoch>`.
- Set loss and val via `set_postfix`.
- The step counter is per-epoch.

On **every validation**, write a persistent line with `tqdm.write` so the validation history stays readable above the moving bar. Include epoch, step, elapsed time, train loss, val loss, and **diff = val − train**, the generalization gap.

```
e2 | step  1350/3000 | 09:13 | loss 2.431 | val 2.512 | diff +0.081
e2 | step  1500/3000 | 10:15 | loss 2.398 | val 2.489 | diff +0.091
```

**Align the digits.** Use fixed-width fields and fixed decimal places (`step {step:>5}`, `loss {loss:6.3f}`, `diff {diff:+6.3f}`, zero-padded times) so consecutive lines form clean columns. Consistent alignment is easier for the user to read.

## Checkpoints

- Path: `checkpoints/<model>-<training>/<seed>-<step|current>.safetensors`.
- **Everything is safetensors, never `.pt`.** Safetensors files are safe to load, mmap-able, and portable. Non-tensor state (step counters, RNG states, config snapshot) goes in a sidecar JSON with the same stem (`<step|current>.json`).
- **Keep the 3 best checkpoints by default**, ranked by the run's headline metric, plus `current`. Prune the rest as training proceeds.

## Device discipline

Which devices matter depends on the project's compute setup (see the suggested questions in the skill), so never assume:

- Use one `get_device()` helper. Never write a literal `.cuda()` or `"cuda:0"` in model or data code.
- **On CUDA, pick a free GPU via `nvidia-smi`** (memory near empty, no compute processes) before binding. Training machines are often shared, and landing on an occupied GPU slows both jobs or causes one of them to run out of memory. Put the check in `get_device()` so every script gets it.
- Guard non-portable features (`torch.compile`, bf16, flash attention) behind the variation definitions so invalid device combinations fail at variation selection.

## Testing discipline

Silent shape bugs are the signature failure of training code. Broadcasting makes wrong shapes run without crashing.

- Assert shapes at function boundaries where a mistake would propagate silently, for example `assert logits.shape[:2] == labels.shape`, with a comment naming the expected shape `(B, T, V)`.
- Comment expected shapes on non-obvious tensor ops.
- The single most bug-prone spot in LLM training is **label/input alignment and masking**: shift-by-one for causal LM, prompt masking with `-100` for SFT, and padding masks.

Custom modules and architectures require more care, especially ones that use einsum. Write a more detailed behavioural test for them.

## Common training pitfalls

Training runs cost time and money. Double check these before a real run starts:

- **Recompilation.** No recompilation should happen during training.
- **Norms.** Torch has built-in norm modules for most common norms. Use them, since they have fused ops that cannot be implemented in pure Python.
- **Forward caching.** Autograd is not perfect. Modules that use `torch.cat` can sometimes cause recomputation.
