---
name: dafny-set
description: "Use when writing or reading Dafny set types (set<T>, iset<T>), or building a set with a comprehension (set x | P :: e, multi-variable, or `<-` domains) — finite and infinite unordered collections with union, difference, intersection, and comprehension."
tags:
  - dafny
  - type
  - collection
related_skills:
  - dafny-seq
  - dafny-multiset
  - dafny-let-and-comprehensions
  - dafny-collection-operators
---

# Dafny: Set

`set<T>` is a finite unordered collection; `iset<T>` is a potentially-infinite ghost variant.

## Syntax

```dafny
var s: set<int> := {1, 2, 3};
assert 2 in s;
var s2 := s - {2};                    // difference, {1, 3}

var u := s + {4};                     // union
var i := s * {2, 3, 9};               // intersection, {2, 3}

ghost var all: iset<nat> := iset x: nat | x > 0;   // infinite set
```

## Set comprehension

`set x: T | P(x) :: e` collects `e` for every `x` of type `T` satisfying `P`.
The `:: e` map expression is optional when there is exactly one variable (it
defaults to the variable itself). Use `iset` for a possibly-infinite result.

```dafny
const c1 := set x: nat | x < 100;                 // {0..99}         (:: x implied)
const c2 := set x: nat | x < 100 :: x * x;        // {0,1,4,...,9801}
const c3 := set x: nat, y: nat | x < y < 100 :: x * y;   // multi-variable
ghost const c4 := iset x: nat | x > 100;          // infinite -> must be iset
const c6 := set x <- c3 :: x + 1;                 // draw x from an existing set

// pairs: yields {(0,1), (0,2), (1,2)}
var pairs := set x: nat, y: nat | x < y < 3 :: (x, y);
```

- Types are optional (inferred); each variable may have a `<- C` domain
  (default: all values of its type) and a `| P` filter (default: `true`).
- For a plain `set`, Dafny must **prove the result finite** or it rejects the
  comprehension; `iset` lifts that requirement (ghost/spec only).
- Comprehensions over reference types (`set o: object | ...`) are allowed only
  in ghost *statements*, not ghost functions, and range over currently
  **allocated** objects.

## Notes

- `set<T>` is finite; `iset<T>` is potentially-infinite and usable only in ghost/spec code.
- Binding order (weakest→strongest): `!!` disjointness, `+` union, `-` difference, `*` intersection.
- `A !! B` means disjoint (`A * B == {}`); it chains: `A !! B !! C` = all mutually disjoint.
- Relations chain like arithmetic: `<` proper subset, `<=` subset, `>=` superset, `>` proper superset.
- Membership with `in` / `!in` (no space in `!in`).
- `|s|` gives cardinality — only valid for finite `set`, not `iset`.

## Extensionality

Two sets are equal iff they contain the same elements. Dafny knows this, but in
an induction where the two sides are *built differently* it often won't apply
extensionality on its own — assert the equality to make it fire.

```dafny
assert s == t;                       // Dafny discharges this via set extensionality

// If it still won't, supply the elementwise proof with assert-by:
assert s == t by {
  forall x ensures x in s <==> x in t { }
}
```
