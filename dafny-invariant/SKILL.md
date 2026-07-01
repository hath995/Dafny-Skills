---
name: dafny-invariant
description: Use when writing or reading Dafny loop invariants (invariant) — a condition that holds before the loop and after every iteration.
tags:
  - dafny
  - specification
  - proof
related_skills:
  - dafny-loops
  - dafny-decreases
  - dafny-requires-ensures
---

# Dafny: invariant

`invariant` states a property that is true on loop entry and re-established after each iteration, letting Dafny reason about the loop.

## Syntax

```dafny
method Zero(a: array<int>)
  modifies a
{
  var i := 0;
  while i < a.Length
    invariant 0 <= i <= a.Length
    invariant forall k :: 0 <= k < i ==> a[k] == 0
    decreases a.Length - i
  {
    a[i], i := 0, i + 1;
  }
}
```

## Notes

- Dafny needs invariants to prove anything about a loop; the loop body is otherwise opaque.
- Each invariant must hold before the loop and be preserved by every iteration.
- Multiple `invariant` clauses are allowed and are conjoined.
- Commonly paired with `decreases` to also prove termination.
