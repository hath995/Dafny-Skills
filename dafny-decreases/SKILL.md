---
name: dafny-decreases
description: Use when writing or reading Dafny termination metrics (decreases) on recursive functions/methods and loops — proves termination to the verifier.
tags:
  - dafny
  - specification
  - proof
related_skills:
  - dafny-loops
  - dafny-lemma
  - dafny-function-and-predicate
---

# Dafny: decreases

`decreases` gives a metric that strictly decreases on each recursive call or loop iteration, proving the code terminates.

## Syntax

```dafny
function Fib(n: nat): nat
  decreases n
{
  if n < 2 then n else Fib(n - 1) + Fib(n - 2)
}

method Count(n: nat, a: array<int>)
{
  var i := 0;
  while i < n
    decreases n - i
  {
    i := i + 1;
  }
}

// Opt out of termination checking entirely
method RunForever()
  decreases *   // allows possible non-termination
{
  while true
    decreases *
  {
    print "still running\n";
  }
}
```

## Notes

- The metric must strictly decrease each step and be bounded below (e.g. a `nat` or a well-founded ordering).
- Often inferred automatically; supply it explicitly when Dafny cannot guess or guesses wrong.
- Can be a lexicographic tuple: `decreases x, y` decreases when `x` drops, or `x` equal and `y` drops.
- Use `decreases *` on methods (and loops inside them) to opt out of termination checking entirely. Not allowed on functions—functions must always terminate. Required for infinite loops and concurrent code.
