# Verification — Rust CLI specifics

Evidence rules and the ladder concept (climb as far as the change warrants,
paste real output, state skipped rungs) are defined in dev-workflow. These
are the concrete rungs for Rust CLIs.

## The rungs

**1. Format, lint & compile — every change.**
```
cargo fmt && cargo clippy --all-targets -- -D warnings
```
Clippy warnings are errors here; fix them, don't `#[allow]` them away
without a stated reason.

**2. Tests — every change touching testable logic.**
```
cargo test
```
Full suite, output shown.

**3. Run the real binary — every change, always.**
`cargo run` (or the release binary) with real arguments exercising the
changed behavior — not just `--help` — driven from throwaway scripts in
`./tmp/<step>/` where setup is involved. Read the actual output. A CLI change
is not verified until the tool has been invoked as a user would invoke it.
Cover the failure path too: feed it the bad input the change is supposed to
handle and confirm the error message and non-zero exit.

**4. Interface-change check — when flags, output format, or exit codes
changed.**
The interface is the contract. Diff old vs new output for the affected
invocations; anything that could break a consumer (a script, a pipe, an
LLM prompt built on this output) gets flagged to the user explicitly — it's
their call, not a silent side effect.

## Silent killers — the CLI self-review checklist

Per dev-workflow's self-review, hunt these in the full diff before
presenting:

- Stream discipline: errors anywhere but stderr? non-errors leaked onto
  stderr? leftover debug prints?
- If the tool has `--json`: do both the success and error paths still emit
  one parseable JSON object?
- `unwrap()`/`expect()` on a user-reachable path?
- Error paths that print but still exit 0?
- Flag renames/removals or output format drift that breaks existing
  consumers?
- Paths that assume this machine (hardcoded home dirs, macOS-only
  locations) in a tool that might run elsewhere?
