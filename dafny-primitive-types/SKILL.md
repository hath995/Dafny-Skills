---
name: dafny-primitive-types
description: Use when writing or reading Dafny primitive types (int, nat, real, bool) — numeric and boolean value types with static constraints.
tags:
  - dafny
  - type
related_skills:
  - dafny-bitvectors
  - dafny-as-conversion
  - dafny-newtype-and-subset
---

# Dafny: Primitive Types

The core scalar types: unbounded integers, naturals, exact reals, and booleans.

## Syntax

```dafny
var i: int := -42;      // unbounded, may be negative
var n: nat := 0;        // nat >= 0 enforced statically
var r: real := 1.5;     // exact rational, needs a decimal point

var b: bool := true;
assert b || !b;         // law of excluded middle
```

## Notes

- `nat` is a subset type of `int` constrained to `>= 0`; the constraint is checked at compile time.
- `int` is arbitrary-precision (no overflow).
- `real` is an exact rational number, not floating point; literals require a decimal point (`1.0`, not `1`).
- `bool` supports `&&`, `||`, `!`, `==>` (implies), `<==>` (iff).
- Convert between numeric types with `as`, e.g. `n as int`, `r as int` (truncates).
