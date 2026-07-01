---
name: dafny-function-by-method
description: Use when writing or reading Dafny functions with imperative bodies (function ... by method) — pairs a functional spec with an efficient loop Dafny proves equivalent.
tags:
  - dafny
  - declaration
  - specification
related_skills:
  - dafny-function-and-predicate
  - dafny-method
  - dafny-loops
---

# Dafny: Function by Method

`function ... by method` gives a function a fast imperative implementation that Dafny verifies matches the functional definition.

## Syntax

```dafny
function Sum(n: nat): nat
{
  if n == 0 then 0 else n + Sum(n - 1)
} by method {
  var r := 0;
  for i := 0 to n {
    r := r + i;
  }
  return r;
}
```

## Notes

- The functional body is the specification used in proofs and reasoning.
- The `by method` body is the code that actually runs (typically iterative and efficient).
- Dafny verifies the method returns exactly what the function specifies.
- Callers get proof-friendly semantics plus imperative performance.
