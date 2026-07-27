---
name: dafny-requires-ensures
description: Use when writing or reading Dafny preconditions and postconditions (requires, ensures) — declares a method's or function's input obligations and output guarantees.
tags:
  - dafny
  - specification
related_skills:
  - dafny-method
  - dafny-function-and-predicate
  - dafny-invariant
---

# Dafny: requires / ensures

`requires` states a precondition the caller must establish; `ensures` states a postcondition the callee must guarantee.

## Syntax

```dafny
method Rot90(p: Point) returns (q: Point)
  requires p != null
{
  // p != null may be assumed here
}

method max(a: nat, b: nat) returns (m: nat)
  ensures m >= a
  ensures m >= b
{
  if a > b { m := a; } else { m := b; }
}
```

## Notes

- Multiple clauses allowed: repeat the keyword or join with `&&` (`ensures m >= a && m >= b`).
- `requires` is the caller's obligation, verified at each call site; inside the body it is assumed.
- `ensures` is the callee's guarantee, verified against the body; callers may assume it after the call.
- Clauses may reference parameters, results, and (in `ensures`) `old(...)` two-state values.
- See dafny-frames for heap reasoning predicates: `fresh()`, `unchanged()`, `allocated()`.
