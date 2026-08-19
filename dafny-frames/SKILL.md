---
name: dafny-frames
description: Use when writing or reading Dafny frame clauses (reads, modifies) and heap reasoning expressions (fresh, unchanged, allocated, old) — declares which heap objects a function may read or method may write, and reason about allocation state. Also triggers on the verifier errors "insufficient reads clause to invoke function" and "assignment may update an object not in the enclosing context's modifies clause".
tags:
  - dafny
  - specification
  - heap
related_skills:
  - dafny-function-and-predicate
  - dafny-method
  - dafny-array
  - dafny-class-and-constructor
  - dafny-twostate
---

# Dafny: reads / modifies

`reads` is a function's read frame (heap it may inspect); `modifies` is a method's write frame (heap it may mutate).

## Syntax

```dafny
method Reverse(a: array<int>)
  modifies a
{
  // permitted to assign a[i] because a is in the modifies frame
}

predicate Sorted(a: array<int>)
  reads a
{
  forall k :: 0 <= k < a.Length - 1 ==> a[k] <= a[k + 1]
}

class Counter {
  var count: int
  method Inc()
    modifies this
  { count := count + 1; }
}
```

## Heap reasoning expressions

`fresh(x)` — reference was newly allocated in the current method call.
`unchanged(obj)` — object's state is identical to pre-call state.
`allocated(e)` — `e` is allocated in the current heap state.
`old(e)` — value of `e` at the start of the current method/function call.

```dafny
class Counter {
  var count: int

  constructor() { count := 0; }

  // Returns a fresh copy without mutating this object.
  method Clone() returns (c: Counter)
    ensures fresh(c) && allocated(c)     // c is newly allocated
    ensures c.count == old(this.count)   // old() reads pre-call state
    ensures unchanged(this)              // this object was not mutated
  {
    c := new Counter();
    c.count := this.count;
  }
}

class XoroShift128Plus {
  var s0: bv64
  var s1: bv64

  constructor(s0: bv64, s1: bv64)
    ensures fresh(this)   // this was just allocated
  {
    this.s0 := s0;
    this.s1 := s1;
  }

  method unsafeNext() returns (out: bv64)
    modifies this
  {
    s1 := s1 ^ s0;
    out := s0 + s1;
  }

  // Advances a *copy*, leaving this generator untouched.
  method next() returns (out: bv64, c: XoroShift128Plus)
    ensures fresh(c)          // c is freshly allocated
    ensures unchanged(this)   // this object was not mutated
  {
    c := new XoroShift128Plus(s0, s1);
    out := c.unsafeNext();
  }
}
```

## Notes

- Frames list the objects/arrays touched: an array `a`, `this`, a set, or `a` plus others.
- Without `modifies a` Dafny rejects writes like `a[i] := ...`; without `reads a` it rejects reads like `a[i]`.
- `modifies this` permits writing this object's fields.
- Anything not in the frame is guaranteed unchanged, which callers rely on.
- `fresh(x)` proves `x` was allocated during this call (not null, not previously reachable). Commonly used in constructor and factory method postconditions.
- `unchanged(obj)` guarantees object fields are identical to pre-call state; useful for pure-like methods returning new objects without mutating `this`.
- `old(e)` captures value at method entry; see dafny-twostate for two-state lemmas/predicates/functions with `old`.
