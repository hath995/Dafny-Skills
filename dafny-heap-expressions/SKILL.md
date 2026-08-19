---
name: dafny-heap-expressions
description: Use when writing or reading Dafny two-state heap expressions (old(), fresh(), unchanged(), allocated()) — reason about pre-state values, allocation, and unchanged heap in specs. Also triggers on the resolver errors "old expressions are allowed only in specification and ghost contexts", "fresh expressions are allowed only in specification and ghost contexts", and "unchanged expressions are allowed only in specification and ghost contexts".
tags:
  - dafny
  - specification
  - heap
related_skills:
  - dafny-frames
  - dafny-twostate
  - dafny-class-and-constructor
---

# Dafny: old / fresh / unchanged / allocated

Two-state expressions that relate a method's pre-state and post-state heap in specifications.

## Syntax

```dafny
class C {
  var count: int
  var f: int

  method Inc()
    modifies this
    ensures count == old(count) + 1   // old = value in the pre-state
  { count := count + 1; }
}

method Make() returns (r: C)
  ensures fresh(r)                     // allocated during this call
{ r := new C; }

method Keep(o: C, a: array<int>)
  ensures unchanged(o)                 // no field of o changed
  ensures unchanged(o.f)               // this specific field unchanged
  ensures unchanged(a[..])             // no array element changed
{ }

twostate lemma L(o: C)
  requires old(allocated(o))           // o existed in the earlier state
  ensures allocated(o)
{ }
```

## Notes

- `old(e)` evaluates `e` in the method's entry heap; useful in `ensures` to compare before/after.
- `fresh(r)` means `r` was not allocated in the pre-state (newly created by the call).
- `unchanged(...)` asserts objects, fields, or array ranges kept their pre-state values.
- `allocated(o)` tests whether `o` exists in a given heap state; common in `twostate` lemmas.
