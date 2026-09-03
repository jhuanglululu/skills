---
name: python-llm-training
description: Domain-specific workflow for Python LLM training projects. Use this from the first message of the task, before writing any code or doing any planning.
---

# Python LLM Training

This skill is general guidance, not law. When the user's prompt says otherwise, the prompt wins.

**This skill extends `dev-workflow`.** Invoke that skill too if it isn't already loaded. This skill supplies the specifics of training an LLM.

## Ground rules

- **uv only.** Use `uv run`, `uv add`, and `uv sync`. Never use bare `pip install` or manually activated venvs. Run `uv init` if the project is not initialized.

## Suggested questions

See the `interviewing` skill first for question format.

1. Where does training happen?
2. Where does inference happen?
3. What is the target language?
4. What is the model size budget?
5. What features do we need? For example: resumability, exporting to pure weights, an MLX implementation.

Summarize and record the decisions. If a later checkpoint, record, or user prompt suggests a decision has changed, ask for clarification and update the recorded decisions.

## Project layout

Typical shape. Adapt to what an existing repo already uses.

```
project/
├── pyproject.toml
├── scripts/          # thin entry points: train.py, eval.py
├── src/<pkg>/        # data.py, model.py, variations.py, ...
├── tests/
├── datasets/
├── records/          # records/<model>-<training>/<seed>-record.jsonl (root-gitignored)
├── checkpoints/      # checkpoints/<model>-<training>/<seed>-<step|current>.safetensors (root-gitignored)
├── tmp/              # scratch check scripts, local-only, contains a "*" .gitignore
└── context/          # local-only working memory (see the context-folder skill)
```

## Default stack

Beyond torch, every project gets these by default via `uv add`:

- **tokenizers**: a tokenizer JSON file is provided in most cases. Load it with the Hugging Face `tokenizers` library, and check for special tokens with a one-time script first.
- **safetensors**: all weights on disk. See the checkpoints section of the training reference.
- **tqdm**: training progress display. See the progress section of the training reference.
- **numpy**: suppresses torch's missing-numpy warning.

## Inference scripts

Use one CLI shape for all inference entry points:

- **Model:** a checkpoint path or a `--model` variation name.
- **Sampling:** `--top-k`, `--top-p`, `--rep-pen`, and `--seed`, each with a sensible default so a bare invocation generates without further flags.
- **Print decode speed** in tokens per second after generating. Skip prefill measurement. It is hard to measure accurately in a simple script, and a misleading number is worse than none.

## References

| Reference | Read it when |
|---|---|
| `references/training.md` | Before writing training code: variations, records, progress display, checkpoints, device selection, testing, pitfalls |
| `references/readme.md` | Before creating or editing the project README |
