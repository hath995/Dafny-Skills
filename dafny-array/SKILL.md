---
name: dafny-array
description: Use when writing or reading Dafny arrays (array<T>, array2<T>, new T[n], new T[m, n], .Length, .Length0, .Length1) — heap-allocated fixed-size 1-D and 2-D arrays.
tags:
  - dafny
  - type
  - oo
  - collection
related_skills:
  - dafny-frames
  - dafny-seq
  - dafny-forall-statement
---

# Dafny: Array

Arrays are heap-allocated, fixed-size reference types. 1-D is `array<T>`, 2-D is `array2<T>`.

## Syntax

```dafny
// 1-D array
method Demo1() {
  var a := new bool[2];
  a[0], a[1] := true, false;
  assert a.Length == 2;
}

method Find(a: array<int>, v: int) returns (index: int)
  reads a
{
  index := 0;
  while index < a.Length {
    if a[index] == v { return index; }
    index := index + 1;
  }
  return -1;
}

// 2-D array
method Demo2() {
  var a := new int[3, 4];
  a[0, 1] := 5;
  assert a.Length0 == 3 && a.Length1 == 4;
}
```

## Notes

- Arrays are reference types allocated on the heap with `new T[n]` (1-D) or `new T[m, n]` (2-D).
- Use `.Length` for a 1-D array; use `.Length0` (rows) and `.Length1` (columns) for a 2-D `array2<T>`.
- Reading elements requires the array in a `reads` frame; writing elements requires it in a `modifies` frame (see dafny-frames).
- Indices are `0`-based; access is `a[i]` for 1-D and `a[i, j]` for 2-D.
