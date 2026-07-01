---
name: dafny-function-and-predicate
description: Use when writing or reading Dafny pure definitions (function, predicate) — side-effect-free expression-bodied definitions; predicate returns bool.
tags:
  - dafny
  - declaration
  - specification
related_skills:
  - dafny-requires-ensures
  - dafny-frames
  - dafny-function-by-method
---

# Dafny: function / predicate

`function` defines a pure, expression-bodied value; `predicate` is a function whose result type is `bool`.

## Syntax

```dafny
function min(a: nat, b: nat): nat
  ensures min(a,b) <= a && min(a,b) <= b
{
  if a < b then a else b
}

function max(a:nat, b: nat): (r: nat) 
  ensures r >= a && r >= b
{

}

predicate win(a: array<int>, j: int)
  reads a
{
  0 <= j < a.Length
}
```

## Notes

- Bodies are single expressions, not statement blocks (use `if/then/else`, not `if { }`).
- A function/predicate that reads heap state needs a `reads` frame (see dafny-frames).
- `predicate P(...)` is exactly `function P(...): bool`.
- In Dafny 4 the old `function method` vs `function` split is gone: functions are compiled by default; use `ghost function` for spec-only.
