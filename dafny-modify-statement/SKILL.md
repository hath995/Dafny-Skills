---
name: dafny-modify-statement
description: Use when writing or reading Dafny modify statements (modify obj) — temporarily allows any field of an object to change, breaking verification knowledge about its state.
tags:
  - dafny
  - statement
  - heap
related_skills:
  - dafny-frames
  - dafny-class-and-constructor
---

# Dafny: Modify Statement

`modify obj;` makes the fields of `obj` unknown to the verifier — it is a controlled havoc that lets external code mutate an object. Used when calling unverified code or simulating non-determinism.

## Syntax

```dafny
class MyClass {
  var x: int

  method N()
    modifies this
  {
    x := 18;
    modify this;              // after this, verifier knows nothing about x
    assert x == 18;           // verification error — cannot conclude this
    assert x >= 0 || x < 0;   // OK — tautology holds regardless of value
  }
}
```

## Notes

- `modify obj;` makes all fields of `obj` unspecified (havoc'd) from the verifier's perspective.
- The object must be in the current method's modifies frame.
- Unlike `havoc x`, which assigns arbitrary values to specific variables, `modify` targets heap objects.
- Useful when interfacing with unverified code that may mutate shared state.
