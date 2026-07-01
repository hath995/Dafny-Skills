---
name: dafny-bitvectors
description: Use when writing or reading Dafny bitvector types (bv8, bv16, bv32, bv64) — fixed-width bit values with bitwise ops and shifts.
tags:
  - dafny
  - type
related_skills:
  - dafny-primitive-types
  - dafny-as-conversion
---

# Dafny: Bitvectors

Fixed-width unsigned bit vectors supporting bitwise arithmetic and shifts.

## Syntax

```dafny
var b: bv8 := 0xFF;
var y := b & 0x0F;        // bitwise AND
var z := b >> 2;          // right shift

assert b as int == 255;   // convert to int
var back := 200 as bv8;   // convert int to bitvector
```

## Notes

- `bvN` is a fixed-width type (`bv8`, `bv16`, `bv32`, `bv64`, and any `bvK`); arithmetic wraps modulo 2^N.
- Supports bitwise `&`, `|`, `^` and shifts `<<`, `>>`.
- Convert to `int` with `as int`; convert an `int` to a bitvector with `as bv8` (value must fit).
- Hex literals (`0xFF`) and decimal literals are both accepted.
