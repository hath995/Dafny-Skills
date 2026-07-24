---
name: dafny-seq
description: Use when writing or reading Dafny sequence types (seq<T>), or building a sequence from a length and index function (seq(n, f) comprehension) — ordered indexable collections with concatenation, slicing, and comprehension.
tags:
  - dafny
  - type
  - collection
related_skills:
  - dafny-set
  - dafny-multiset
  - dafny-slicing
  - dafny-collection-operators
---

# Dafny: Seq

`seq<T>` is an immutable 0-indexed ordered sequence of elements.

## Syntax

```dafny
var q: seq<int> := [1, 4, 9, 16];
assert q[2] == 9;             // index
assert q[1..3] == [4, 9];     // slice, j exclusive

var r := q + [25];            // concatenate (NOT ++)
```

## Sequence comprehension (`seq(n, f)`)

Build a sequence from a length and an index function `f: nat -> T` — Dafny's
equivalent of a sequence comprehension.

```dafny
// [0, 1, 4, 9, 16] — element i is f(i)
var squares := seq(5, i => i * i);

// Mapping over an existing sequence: the lambda must carry a `requires`
// bound so Dafny knows every index it reads is in range.
var doubled := seq(|q|, i requires 0 <= i < |q| => q[i] * 2);

// Works for any element type — e.g. build a seq<char> from a seq of regs.
var text := seq(|value|, i requires 0 <= i < |value| => RegToChar(value[i]));
```

## Notes

- 0-indexed and ordered; index with `q[i]`.
- Concatenate with `+` — NOT `++`.
- Slices: `q[i..j]`, `q[..j]` (prefix), `q[i..]` (suffix).
- `|q|` gives the length.
- Membership with `in` / `!in`.
- Update a position functionally with `q[i := v]` returning a new sequence.
- Build one with `seq(n, f)` where `f: nat -> T` maps each index; to read another collection inside `f`, add `requires 0 <= i < n` to the lambda (see the comprehension example above).

## Pitfalls

### Sequence comprehensions fail for map elements

`[x | x in s :: pred(x)]` triggers **"rbracket expected"** when `T` is a `map<K,V>` type. Dafny cannot parse the `|` separator inside bracket comprehensions for map-typed elements.

**Workaround:** use recursive helper functions:

```dafny
function SeqFilter(s: seq<Record>, pred: (Record) -> bool): seq<Record>
  decreases |s|
{
  if |s| == 0 then []
  else if pred(s[0]) then [s[0]] + SeqFilter(s[1..], pred)
  else SeqFilter(s[1..], pred)
}

function SeqMap(s: seq<Record>, f: (Record) -> Record): seq<Record>
  decreases |s|
{
  if |s| == 0 then []
  else [f(s[0])] + SeqMap(s[1..], f)
}
```

### `decreases` clause placement

For expression-bodied functions, `decreases` goes after the return type, before `{`:

```dafny
function F(s: seq<T>): int decreases |s| { ... }  // correct
// NOT: function F(s: seq<T>): int { ... decreases |s| }
```

Two sequences are equal iff they have the same length and equal elements at every
index. When the sides are assembled differently (e.g. `f(xs) + [y]` vs `[x] + f(ys)`
in an induction), the prover usually needs the equality asserted.

```dafny
assert ss == ts by {
  assert |ss| == |ts|;
  forall i | 0 <= i < |ss| ensures ss[i] == ts[i] { }
}

// A very common induction bridge — split at k and rejoin:
assert xs == xs[..k] + xs[k..];
// slicing by first or last element
assert xs == [xs[0]] + xs[1..];
assert xs == xs[..|xs|-1]+[xs[ |xs| - 1 ]];
//slice equalities
assert xs[..i+1][..i] == xs[..i];
assert xs[..i+1][i] == xs[i];
assert (xs + ys)[0] == xs[0];
assert (xs + ys)[1..] == xs[1..] + ys;
```
