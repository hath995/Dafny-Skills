---
name: dafny-map
description: Use when writing or reading Dafny map types (map<K,V>, imap<K,V>) — finite and infinite key-value associations with update and domain queries.
tags:
  - dafny
  - type
  - collection
related_skills:
  - dafny-set
  - dafny-let-and-comprehensions
  - dafny-collection-operators
---

# Dafny: Map

`map<K,V>` is a finite key-value association; `imap<K,V>` is a potentially-infinite ghost variant.

## Syntax

```dafny
var m: map<int,string> := map[1 := "one", 2 := "two"];
assert 1 in m;                // key-domain test
m := m[3 := "three"];         // update / extend

var keys := m.Keys;           // set of keys
var vals := m.Values;         // set of values

ghost var sq: imap<nat,nat>;  // infinite map (spec only)
```

## Notes

- `map` is finite; `imap` is potentially-infinite and usable only in ghost/spec code.
- Update or extend with `m[k := v]` (functional, returns a new map).
- `k in m` tests membership in the key domain.
- `m.Keys` / `m.Values` return sets; `m[k]` reads the value (requires `k in m`).
- `|m|` gives the size — finite maps only.
- Map comprehension: `map k | 0 <= k < 3 :: k * k`.

## Extensionality

Two maps are equal iff they have the same key set and equal values on those keys.
In inductive proofs the prover frequently needs this spelled out rather than
inferring it across two differently-constructed maps.

```dafny
assert m == n by {
  assert m.Keys == n.Keys;
  forall k | k in m ensures m[k] == n[k] { }
}
```
