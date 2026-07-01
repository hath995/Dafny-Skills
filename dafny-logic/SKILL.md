---
name: dafny-logic
description: Use when writing or reading Dafny logical expressions (!, &&, ||, ==>, <==, <==>, forall, exists) — boolean connectives and quantifiers in specs and assertions.
tags:
  - dafny
  - specification
  - expression
  - proof
related_skills:
  - dafny-quantifiers
  - dafny-assert-assume
  - dafny-requires-ensures
  - dafny-forall-statement
---

# Dafny: logical connectives and quantifiers

Boolean operators and quantifiers used inside specs, assertions, and function bodies.

## Syntax

```dafny
method Logic(x: int, y: int, z: bool, a: bool, b: bool, arr: array<int>, j: int)
{
  assume (z || !z) && x > y;                 // not, or, and
  assert j < arr.Length ==> arr[j] * arr[j] >= 0;  // implies
  assert !(a && b) <==> !a || !b;            // iff (De Morgan)

  assume forall n: nat :: n >= 0;            // universal
  assert forall k :: k + 1 > k;              // type inferred as int
  assert exists k :: k > 100;                // existential
}
```

## Notes

- `==>` is implies, `<==` is explies (reverse implication), `<==>` is iff.
- `==>` is right-associative and short-circuits; `&&` / `||` short-circuit left-to-right.
- Quantifier form is `forall vars :: body` / `exists vars :: body`; the body follows `::`.
- Quantifier variable types can be annotated (`k: nat`) or inferred from the body.
