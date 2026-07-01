---
name: dafny-iterator
description: Use when writing or reading Dafny generators/coroutines (iterator, yields, yield, MoveNext) — emits a sequence of values one at a time.
tags:
  - dafny
  - declaration
  - oo
related_skills:
  - dafny-loops
  - dafny-method
---

# Dafny: Iterator

An `iterator` is a coroutine that produces a stream of values via `yield`.

## Syntax

```dafny
iterator Range(lo: int, hi: int) yields (i: int)
{
  var k := lo;
  while k < hi {
    yield k;      // emit one value, then resume here
    k := k + 1;
  }
}
```

## Notes

- `yields (i: int)` declares the type/name of the produced values.
- Each `yield` emits one value and suspends until the next step.
- Iterators are heap-allocated objects driven by a `MoveNext()` protocol.
- Callers create the iterator, then repeatedly call `MoveNext()` reading the yielded field.
