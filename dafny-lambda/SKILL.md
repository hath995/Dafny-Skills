---
name: dafny-lambda
description: Use when writing or reading Dafny lambda expressions ((x) => expr, (x: T, y: T) => expr, or a lambda with a requires clause) — anonymous total functions you can apply and pass around.
tags:
  - dafny
  - expression
related_skills:
  - dafny-let-and-comprehensions
  - dafny-function-and-predicate
---

# Dafny: Lambda Expressions

Anonymous function values written with `=>`. They can carry `requires`/`reads` clauses and are applied like ordinary functions.

## Syntax

```dafny
// single parameter
var f := (x: int) => x * x;
assert f(3) == 9;

// multiple parameters
var g := (x: int, y: int) => x + y;
assert g(2, 5) == 7;

// lambda with a requires clause, used inside seq(...)
method PrintRegs(value: seq<int>) {
  print seq(|value|, i requires 0 <= i < |value| => RegToChar(value[i]));
}
```

## Notes

- A lambda is an anonymous total function value; its type is `T -> U` (e.g. `int -> int`).
- Applied with normal call syntax: `f(3)`.
- May carry a `requires` clause to constrain inputs (making it partial over that precondition) and a `reads` clause if it reads heap state.
- The `i requires 0 <= i < |value| => ...` form gives the lambda a precondition Dafny uses to justify the indexing `value[i]`.
- Parameter types may be omitted when inferable from context.
