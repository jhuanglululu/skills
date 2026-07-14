# Systematic Debugging

The discipline: **reproduce small, look at actual state, form a hypothesis,
test the hypothesis — no fixes before a diagnosis.** Stacked speculative
fixes obscure the bug, corrupt the evidence, and in expensive-to-run domains
each guess has a real cost.

## The investigation order

1. **Reproduce at minimum scale.** Shrink the failing case until it fails
   fast and locally — smallest input, fewest components, cheapest
   environment. A bug that *disappears* when shrunk is itself a diagnosis
   clue: something about scale, timing, or environment is part of the cause.
   The project-type skill defines what "minimum scale" looks like in its
   domain.
2. **Look at actual state before theorizing.** Print the real values at the
   failure boundary — actual inputs, actual intermediate data, actual
   config — not what the code *should* produce. Most "impossible" bugs are
   an input that isn't what you assumed.
3. **Form a specific, falsifiable hypothesis.** "It fails because X is
   empty" — concrete enough that a single observation can kill it.
4. **Show the hypothesis to the user before testing it.** State what you
   think is happening and what you're about to check. The user runs the
   project interactively and may have seen things you can't see from the
   code — and the original bug description may not match the actual bug.
   This checkpoint catches a wrong premise before any time is spent chasing
   it.
5. **Test the hypothesis directly.** "X is empty" → print X. Every rejected
   hypothesis narrows the search; every untested guess widens it.
6. **Bisect with known-good components.** Swap suspect parts for known-good
   ones (synthetic input, stub dependency, last-known-good commit). Which
   swap fixes it tells you which side the bug is on.
7. **Fix, then prove the fix at the same minimum scale** that reproduced the
   bug. Where feasible, add the assertion or test that would have caught
   it — bugs recur in the places they were born.

## Rationalizations to catch yourself on

- "It's probably X, let me just try changing it" — that's a hypothesis;
  test it by *observation* before changing code.
- "This one's simple, no need for the process" — simple-looking bugs with
  wrong first guesses are how an hour becomes an afternoon.
- "I'll add a few fixes at once to save time" — now a pass/fail tells you
  nothing about which change mattered.
