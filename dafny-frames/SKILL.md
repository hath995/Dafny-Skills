---
name: dafny-frames
description: Use when writing or reading Dafny frame clauses (reads, modifies) — declares which heap objects a function may read or a method may write.
tags:
  - dafny
  - specification
  - heap
related_skills:
  - dafny-function-and-predicate
  - dafny-method
  - dafny-array
---

# Dafny: reads / modifies

`reads` is a function's read frame (heap it may inspect); `modifies` is a method's write frame (heap it may mutate).

## Syntax

```dafny
method Reverse(a: array<int>)
  modifies a
{
  // permitted to assign a[i] because a is in the modifies frame
}

predicate Sorted(a: array<int>)
  reads a
{
  forall k :: 0 <= k < a.Length - 1 ==> a[k] <= a[k + 1]
}

class Counter {
  var count: int
  method Inc()
    modifies this
  { count := count + 1; }
}
```

## Notes

- Frames list the objects/arrays touched: an array `a`, `this`, a set, or `a` plus others.
- Without `modifies a` Dafny rejects writes like `a[i] := ...`; without `reads a` it rejects reads like `a[i]`.
- `modifies this` permits writing this object's fields.
- Anything not in the frame is guaranteed unchanged, which callers rely on.
