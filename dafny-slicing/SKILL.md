---
name: dafny-slicing
description: Use when writing or reading Dafny slice expressions (s[i..j], s[i..], s[..j]) — half-open subsequence, substring, and array slicing.
tags:
  - dafny
  - expression
  - collection
related_skills:
  - dafny-seq
  - dafny-char-and-string
  - dafny-collection-operators
---

# Dafny: Slicing

Extract a contiguous range from a `seq`, `string`, or array using half-open `[i..j)` bounds.

## Syntax

```dafny
var q := [10, 20, 30, 40, 50];

var mid  := q[1..3];   // [20, 30]   indices 1,2
var tail := q[2..];    // [30, 40, 50]
var head := q[..3];    // [10, 20, 30]
var all  := q[..];     // full copy: [10,20,30,40,50]

var a := new int[3];
var asSeq := a[..];    // whole array as a seq
```

## Notes

- Ranges are half-open `[i, j)`: `s[i..j]` includes index `i` up to but not including `j`.
- Omitting the low bound (`s[..j]`) starts at 0; omitting the high bound (`s[i..]`) runs to the end.
- Works on `seq`, `string`, and arrays; `a[..]` converts an array into a `seq`.
- `s[..]` yields a copy of the whole collection.
- Bounds must satisfy `0 <= i <= j <= |s|` or Dafny reports an index-out-of-range error.
