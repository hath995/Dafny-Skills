---
name: dafny-let-and-comprehensions
description: "Use when writing or reading Dafny let expressions (var x := e; body in expression context), set comprehensions (set x | P :: e), map comprehensions (map x | P :: k := v), or seq(n, f) builders — value-producing binding and collection construction."
tags:
  - dafny
  - expression
  - collection
related_skills:
  - dafny-lambda
  - dafny-set
  - dafny-map
  - dafny-seq
---

# Dafny: Let Expressions and Comprehensions

Expression-level binding and comprehension forms that build values without statements.

## Syntax

```dafny
// let expression: binds m, then the body uses it (yields a value)
function F(n: nat): nat { var m := n + 1; m * m }

// set comprehension: {0,1,4,9,...,81}
var s := set x: nat | x < 10 :: x * x;

// map comprehension: {0:0, 1:1, 2:4, 3:9, 4:16}
var m := map x: nat | x < 5 :: x := x * x;

// seq builder: [0, 1, 4, 9, 16]
var q := seq(5, i => i * i);
```

## Notes

- The `var x := e; body` let form is an *expression*, not a statement: it evaluates to `body` with `x` bound. Legal inside functions and other expressions.
- Comprehensions require a bounded/decidable range for the bound variable (here `x: nat` plus `x < N`), so Dafny can enumerate finitely.
- Set comp `set x | P :: e` collects `e` over all `x` satisfying `P`; map comp `map x | P :: k := v` builds key `k` to value `v`.
- `seq(n, f)` builds a length-`n` sequence whose element `i` is `f(i)`; `f` is an index function of type `nat -> T` (may need a `requires` bound, see dafny-lambda).
