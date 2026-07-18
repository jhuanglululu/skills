# Debugging — LLM training specifics

The investigation order (reproduce small → look at actual state → hypothesis
→ user reviews it → test → bisect → fix and prove) is defined in the
debugging skill. This file maps it onto training code and catalogs the
standard failures.

## Applying the investigation order here

- **Minimum scale means:** smoke config, on the development machine, CPU
  if possible — CPU gives better stack traces than MPS/CUDA and removes
  device-specific causes. A bug that disappears at small scale points at fp16/bf16, data
  dependence, or distributed-only causes.
- **"Look at actual state" means the batch.** Decode and print one full
  batch exactly as the model sees it: input tokens as text, labels with
  `-100`s visible, attention mask. The majority of "model won't learn" bugs
  are visible right here — wrong template, missing EOS, labels not shifted,
  everything masked out. Data before model, always.
- **Bisect with:** synthetic data the model *must* fit, or a known-good
  model on your data. Which swap fixes it tells you which side the bug is
  on.
- **Prove the fix at smoke scale** before relaunching anything expensive,
  and add the assertion or unit test that would have caught it.

## The two universal sanity checks

- **Overfit one batch.** Any healthy model + loop must drive loss to ~0 on a
  single repeated batch within a few hundred steps. If it can't, the bug is
  in the loop/loss/labels — not the data, not the hyperparameters. This is
  the fastest end-to-end correctness test of the training path.
- **Loss at step 0 ≈ ln(vocab_size).** A causal LM with random init should
  start near `ln(V)` (~10.8 for V=50k). Materially lower → leakage or
  degenerate labels; materially higher → init or loss-scaling bug.

## Failure catalog

**Loss → NaN/Inf.** Check `grad_norm` in the JSONL just before the blow-up —
a climb before the spike means instability (lower LR, add/lower grad clip,
longer warmup); a sudden spike from calm means a bad batch (inspect the
exact batch by step number) or fp16 overflow (bf16 doesn't have this
problem; suspect fp16 first). Also: division by zero in a custom loss/metric
when a batch has zero unmasked tokens.

**Loss flat from the start.** Almost always data/labels: everything masked,
labels not aligned to inputs, LR = 0 (scheduler bug — check the actual `lr`
value in the JSONL), optimizer not stepping (`zero_grad`/`step` order,
parameters not registered, frozen by mistake — print
`sum(p.requires_grad for p in model.parameters())`).

**Loss decreases but the model is bad at inference.** Train/inference skew:
different prompt template between SFT data and generation, missing/extra
special tokens, wrong dtype at load, or a mismatch between the custom MLX
inference module and the torch model definition — compare MLX output
against torch generation from the same checkpoint on the same prompt before
blaming the checkpoint.

**OOM.** In order of preference: gradient accumulation (halve batch, double
accum — identical math), activation checkpointing, shorter sequences, then
smaller model. On MPS, memory pressure also shows up as extreme slowdown
rather than a clean OOM.

**Val diverging from train (overfitting).** The default failure for small
models — watch the `diff` column (val − train) in the progress display; a
steadily growing gap is the earliest warning. Not a bug in the code but in
the setup: model too large for the data, too many epochs, or data too
repetitive. Remedies in order: more/more-varied data, smaller model or
regularization (dropout, weight decay), stop earlier — the keep-3-best
checkpoint rule means the best-val checkpoint already exists; use it. If
val is *rising from step 0*, that's not overfitting — suspect a data bug
(val set leaked into train makes the opposite pattern; val set from a
different distribution makes this one).

**Steps too slow.** If `torch.compile` is on, check recompilation first
(`TORCH_LOGS=recompiles`) — steady-state recompiles from varying shapes
(unpadded batches, changing sequence lengths) silently destroy throughput;
pad to fixed shapes or mark dims dynamic. Also check the GPU isn't shared
with another process (`nvidia-smi`). Then profile: check `sec_per_step` in
the JSONL and whether GPU utilization is low (dataloader bottleneck — more
workers, pre-tokenize the dataset) or high (the model is just big — it
belongs on the training hardware). Don't micro-optimize the loop without a
measurement.

**Resume produces different results.** Optimizer state or RNG state not in
the checkpoint, or dataloader not fast-forwarded. Checkpoint = model +
optimizer + scheduler + step + RNG states.
