---
name: dafny-collection-operators
description: Use when writing or reading Dafny collection operators (|s| cardinality/length, s + t concat/union, in / !in membership) — core operations over seq, set, multiset, map, and string.
tags:
  - dafny
  - expression
  - collection
related_skills:
  - dafny-seq
  - dafny-set
  - dafny-map
  - dafny-multiset
  - dafny-slicing
---

# Dafny: Collection Operators

Length, combination, and membership operators shared across Dafny collection types.

## Syntax

```dafny
// cardinality / length: |.|
assert |"hi"| == 2;
assert |{1, 2, 3}| == 3;
assert |[1, 2]| == 2;

// + : seq/string concat, set/map/multiset union
assert [1, 2] + [3, 4] == [1, 2, 3, 4];
assert {1, 2} + {2, 3} == {1, 2, 3};

// in / !in : membership (map tests the key domain)
assert 2 in {1, 2, 3};
assert 5 !in [1, 2, 3, 4];
assert "k" in map["k" := 1];
```

## Notes

- `|.|` gives length for `seq`/`string` and cardinality for `set`/`multiset`/`map`.
- `+` concatenates sequences and strings and unions sets, maps, and multisets. Sequence concatenation is `+`, never `++`.
- Set difference is `-` and intersection is `*` (not shown above).
- `in` / `!in` test membership; on a `map` they test whether a value is in the key domain, not the values.
