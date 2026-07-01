---
name: dafny-multiset
description: Use when writing or reading Dafny multiset types (multiset<T>) — unordered collections tracking element multiplicities.
tags:
  - dafny
  - type
  - collection
related_skills:
  - dafny-set
  - dafny-seq
  - dafny-collection-operators
---

# Dafny: Multiset

`multiset<T>` is an unordered collection where each element has a multiplicity (count).

## Syntax

```dafny
var t: multiset<int> := multiset{1, 1, 2};
assert t[1] == 2;             // t[1] is the multiplicity of 1
assert t[9] == 0;             // absent element has multiplicity 0

var u := t - multiset{1};     // remove one occurrence of 1
var w := t[1 := 5];           // set multiplicity of 1 to 5 (new multiset)

var a := multiset{1, 1, 2} + multiset{1, 3};   // union:        1->3, 2->1, 3->1
var b := multiset{1, 1, 2} * multiset{1, 3};   // intersection: 1->1  (min counts)

var q := [3, 3, 3];
var m := multiset(q);         // build from a seq (or a set)
```

## Operators

Binary ops act on multiplicities (weakest→strongest binding: `!!`, `+`, `-`, `*`):

- `+` union — multiplicities **add**.
- `-` difference — multiplicities **subtract**, clamped at 0.
- `*` intersection — multiplicity is the **min** of the two.
- `A !! B` disjointness (`A * B == multiset{}`); chains: `A !! B !! C` = mutually disjoint.
- Relations chain: `<` proper submultiset, `<=` submultiset, `>=` supermultiset, `>` proper supermultiset.

## Notes

- Unordered, but tracks duplicate counts. There is **no comprehension form** —
  build via `multiset{...}` literals or `multiset(q)` from a `seq`/`set`.
- `t[x]` gives the multiplicity of `x` (0 if absent); `t[x := n]` returns a new
  multiset with that multiplicity set to `n`.
- Membership `x in t` means `t[x] > 0`; `|t|` is the total count including duplicates.

## Extensionality

Two multisets are equal iff every element has the same multiplicity. Assert it to
bridge two differently-built multisets — the usual case is proving a sort or
reordering is a permutation of its input.

```dafny
assert a == b by {
  forall x ensures a[x] == b[x] { }
}
```
