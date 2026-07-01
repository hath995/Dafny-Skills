---
name: dafny-twostate
description: Use when writing or reading Dafny two-state constructs (twostate lemma, twostate predicate, twostate function, old) — reason across a previous and current heap state.
tags:
  - dafny
  - declaration
  - specification
  - heap
related_skills:
  - dafny-heap-expressions
  - dafny-lemma
  - dafny-function-and-predicate
---

# Dafny: Twostate

Two-state declarations relate an earlier heap state to the current one via `old(...)`.

## Syntax

```dafny
class Counter { var val: int }

twostate lemma Changed(c: Counter)
  requires old(c.val) < c.val
  ensures c.val > 0
{
}

twostate predicate Increased(c: Counter)
{
  old(c.val) < c.val
}

twostate function Delta(c: Counter): int
{
  c.val - old(c.val)
}
```

## Notes

- `old(...)` denotes a value in the earlier state; the bare expression is the current state.
- Used for framing and history reasoning about how objects changed.
- Available as `twostate lemma`, `twostate predicate`, and `twostate function`.
- See also dafny-heap-expressions for `old`, `fresh`, and framing details.
