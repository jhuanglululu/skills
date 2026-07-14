---
name: rust-cli
description: Domain-specific workflow for Rust CLI binary projects — cargo, clap-based argument parsing, error handling conventions, and CLI verification (clippy clean, real invocations of the built binary). Use whenever the task touches a Rust command-line tool — new binaries, subcommands, flags, output formatting, or changes to existing CLI tools like fetch — from the first message of the task, before writing any code. Always use together with the dev-workflow skill, which owns the general process this skill plugs into.
---

# Rust CLI Binaries

This skill is general guidance, not law: when the user's prompt says
otherwise, the prompt wins.

Domain knowledge for Rust command-line tools. **This skill extends
`dev-workflow`** — invoke that too if it isn't already loaded. Process rules
(task sizing, plan files, debugging discipline, evidence rules, git) live
there; this skill supplies what's different about Rust CLIs.

What's different, in one sentence: **a CLI's interface is its contract** —
flags, output format, and exit codes are what scripts, pipes, and other
tools depend on, so interface changes deserve the scrutiny that API changes
get elsewhere.

## Ground rules

- **cargo for everything**: `cargo add`, `cargo test`, `cargo clippy`.
  Stable toolchain unless the project already pins otherwise.
- **stderr is for errors only; everything else goes to stdout** (output,
  progress, info logging). An optional `--json` mode exists for long,
  machine-parsed output — conventions in `references/implementation.md`.
- **Exit codes mean something**: 0 success, non-zero failure. A CLI that
  prints an error but exits 0 breaks every script that calls it.

## Domain deltas per phase

- **Planning:** mostly per dev-workflow. The domain questions worth asking
  when the repo doesn't answer them: who consumes the output (humans, pipes,
  LLMs — decides formatting)? Is the interface stable (does anything already
  depend on current flags/output)? One-shot tool or long-lived?
- **Implementation:** standard stack (clap, anyhow), thin `main.rs` over a
  testable library, error and output conventions:
  `references/implementation.md`.
- **Debugging:** per dev-workflow, nothing exotic. `RUST_BACKTRACE=1` on
  panics; `dbg!()` beats print-debugging; most "impossible" CLI bugs are
  argument parsing or env differences — check what the binary actually
  received.
- **Verification:** the concrete rungs — fmt/clippy/test, then running the
  actual built binary: `references/verification.md`.

## Reference files

| File | Read it when |
|---|---|
| `references/implementation.md` | Writing/modifying any project code |
| `references/verification.md` | Before claiming any task is done |
| `references/readme.md` | Creating or updating the project README |
