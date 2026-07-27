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
newtype Pos = x: int | x > 0 witness 1   // 0 fails the constraint, so a witness is required

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

## Witness clauses

If Dafny cannot prove a subset type is non-empty, provide a `witness`:

```dafny
// Error — Dafny cannot prove existence:
// type OddInt = x: int | x % 2 == 1

// OK — witness proves the type has at least one value:
type OddInt = x: int | x % 2 == 1 witness 73
```

## Arrow subset types (->, -->, ~>)

Three function arrow types distinguish heap access and preconditions:

- `A ~> B` — partial functions that may read heap state
- `A --> B` — pure functions (no heap reads), but may have preconditions
- `A -> B` — total pure functions (no heap, no preconditions)

Each is a subset of the one above it: every `->` is a `-->`, and every `-->` is a `~>`. Use `f.requires(x)` and `f.reads(x)` to state a function value's precondition and read frame.

```dafny
class Cell { var v: int }

// A ~> B: may read the heap and may be partial
function ApplyReader(f: int ~> int, x: int): int
  requires f.requires(x)
  reads f.reads(x)
{ f(x) }

// A --> B: no heap reads, but may have a precondition
function ApplyPartial(f: int --> int, x: int): int
  requires f.requires(x)
{ f(x) }

// A -> B: total and heap-independent, callable at any argument
function ApplyTotal(f: int -> int, x: int): int
{ f(x) }

method Demo(c: Cell) {
  var total: int -> int := x => x + 1;                       // total
  var partial: int --> int := x requires x != 0 => 100 / x;  // partial
  var reader: int ~> int := x reads c => x + c.v;            // reads the heap
  assert total(3) == 4;
}
```
