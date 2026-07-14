# README — Rust CLI projects

The README is the tool's front door: its audience is someone (or an LLM)
deciding what the tool does and how to invoke it. The interface is the
contract, and the README is where the contract is written down in prose —
**show it with real invocations and real output**, not descriptions.

## Structure

```markdown
# <tool>
One line: what it does. One short paragraph if the one-liner needs it.

## Install
cargo install --path .

## Usage
Real invocations with their actual output:

    $ tool do-thing input.txt
    <the genuine output, pasted>

One example per major mode — including --json (show the JSON shape) and
the main failure case (show the error message and note the non-zero exit).

## Configuration (if any)
Config file location and format, env vars — with a minimal working example.
```

## Rules

- **Examples over prose.** Every claimed flag/mode appears in a runnable
  example; every example's output is genuine, pasted from a real run
  (verification evidence rules apply).
- **The README pins the interface.** When flags, output format, or exit
  codes change (verification rung 4), updating the README is part of that
  change, same commit — the examples are the visible diff of the contract.
- Keep it short: no architecture internals, no dependency tour, no
  changelog. A tool this size deserves a README readable in one screen or
  two.
