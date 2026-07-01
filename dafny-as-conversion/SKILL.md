---
name: dafny-as-conversion
description: Use when writing or reading Dafny type conversions (e as T) — explicit numeric casts and up/down reference conversions.
tags:
  - dafny
  - expression
  - type
related_skills:
  - dafny-primitive-types
  - dafny-bitvectors
  - dafny-newtype-and-subset
---

# Dafny: as Conversion

Explicit type conversion with `as`, used for numeric casts and reference up/down casts.

## Syntax

```dafny
// numeric / bitvector conversion
assert (255 as bv8) == 0xFF;

// nat -> int
var n: nat := 5;
var i: int := n as int;

// subset / newtype require explicit as
newtype u8 = x: int | 0 <= x < 256
var b: u8 := 200 as u8;
```

## Notes

- Converts between numeric types (`int`, `nat`, `real`, `bv`) and up or down between reference types.
- The conversion must be provably valid: the source value must be in range for the target (e.g. within `bv8` or the target `newtype`'s constraint), otherwise Dafny reports a verification error.
- `newtype` and subset types are distinct types, so moving to/from them requires an explicit `as`.
- `as` is a checked conversion, not a reinterpretation; it preserves the numeric value where possible.
