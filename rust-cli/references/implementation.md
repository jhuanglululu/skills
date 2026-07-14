# Implementation — Rust CLI specifics

General rules (match existing conventions, dependencies in the manifest,
risk-based testing principle) are in dev-workflow.

## Standard stack

- **clap** (derive API, with the `wrap_help` feature:
  `cargo add clap --features derive,wrap_help`) for argument parsing —
  subcommands as enums, args as structs, so the CLI surface is readable in
  one place.
- **shuo** (https://crates.io/crates/shuo) — the user's own crate for
  simple logging with info and error. Use it instead of ad-hoc `eprintln!`
  logging or heavier log frameworks. If a feature seems like it would be a
  useful addition to shuo (a new level, formatting option, etc.), **ask the
  user** — they maintain it and would rather grow the crate than work
  around it.
- **anyhow** for application errors, with `.context()` at every fallible
  boundary (file paths, network calls) so failures name what they were
  doing. Reserve **thiserror** for when a library crate needs typed errors.
- Beyond those, stay lean: prefer std until a dependency clearly earns its
  place. Every dependency is compile time and supply-chain surface.

## Structure

- **Thin `main.rs`.** `main()` parses args, calls into the rest of the
  code, turns the resulting `anyhow::Error` into a clean error + non-zero
  exit. These CLIs are mostly small and simple — don't over-architect them
  for testability; correctness is checked end-to-end (see testing below).

## Error and panic conventions

- **No `unwrap()`/`expect()` on any path user input can reach.** A panic
  with a backtrace is an acceptable response to a program bug, never to a
  bad argument, a missing file, or malformed input — those get a clean
  error message on stderr and a non-zero exit.
- Error messages say what failed, with what input, and what to try:
  `failed to read config at ~/.config/fetch.toml: permission denied` beats
  `io error`.
- Handle broken pipes gracefully (`tool | head` must not panic-spam):
  either ignore SIGPIPE-style errors on stdout writes or exit quietly.

## Output conventions

- **stderr is for errors only; everything else goes to stdout** — output,
  progress, info logging (shuo's info included).
- **`--json` is optional, not default** — suggest adding it when it would
  clearly help: output long enough that raw text wastes tokens, or output
  a program/LLM will parse. The rule: `--json` for long output that gets
  parsed easily with Python; plain text for human-read and short output.
- In `--json` mode, **both output and errors are JSON** — one parseable
  object either way, so a Python caller never needs a fallback text parser.
  Exactly the format, nothing more: no headers, no color codes.
- **A progress-disabling flag (e.g. `--no-progress`) is optional, not
  default** — same treatment as `--json`: suggest it when it would clearly
  help, i.e. when the tool emits progress on stdout and its output is
  likely to be piped, where progress lines would contaminate the stream.
- Color/formatting only when stdout is a TTY; respect `NO_COLOR`.

## Shell completions (on request)

When the user asks for tab completion:

- Target **zsh** by default.
- Implement it as a subcommand named **`__completions`** that prints the
  completion script (via `clap_complete`).
- **Hide `__completions` from the completion output itself** (and from
  help): mark the subcommand hidden so a user tab-completing subcommands
  never sees the plumbing.

## Testing, applied to small CLIs

These CLIs are mostly small and simple: **end-to-end invocation usually
tells more than unit tests.** Test by running the built binary against real
inputs from throwaway scripts in `./tmp/<step>/` (per dev-workflow's
testing reference) — real args, real files, read the actual output.

- **Unit tests** only where dev-workflow's risk-based bar is genuinely met:
  logic tricky enough that a bug is easy to introduce and hard to spot by
  reading. clap derive itself never needs testing.
- **Integration tests** (`tests/` with `assert_cmd` or equivalent) only for
  CLIs that have grown an interface contract worth pinning — formats and
  exit codes that other things depend on.
