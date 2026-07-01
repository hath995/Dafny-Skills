---
name: dafny-newtype-and-subset
description: Use when writing or reading Dafny constrained/derived numeric types (newtype, type synonym, subset type, as) — a distinct new type vs. an assignable subset or alias.
tags:
  - dafny
  - declaration
  - type
related_skills:
  - dafny-primitive-types
  - dafny-as-conversion
  - dafny-datatype
---

# Dafny: Newtype and Subset Type

`newtype` creates a distinct incompatible type; a subset `type` (or synonym) shares its base type's operations.

## Syntax

```dafny
newtype byte = x: int | 0 <= x < 256
newtype Pos = x: int | x > 0

type Name = string            // plain synonym
type Pct = x: int | 0 <= x <= 100  // subset type

method Demo()
{
  var b: byte := 5 as byte;   // newtype needs explicit conversion
  var i: int := b as int;
  var p: Pct := 50;           // subset assigns to/from base freely
  var j: int := p;
}
```

## Notes

- `newtype` is a brand-new numeric type incompatible with its base; convert with `as`.
- A subset `type` is assignable to and from its base without any conversion.
- `type Name = T` with no constraint is a plain alias/synonym.
- Key difference: `newtype` is a separate type; a subset `type` reuses the base's operations.
