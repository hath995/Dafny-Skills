---
name: dafny-class-and-constructor
description: Use when writing or reading Dafny classes (class, constructor, static, new, object) — reference types with fields, constructors, static members, and heap allocation.
tags:
  - dafny
  - declaration
  - oo
  - type
related_skills:
  - dafny-method
  - dafny-trait
  - dafny-array
---

# Dafny: Class and Constructor

A class is a reference type with mutable fields, constructors, and instance/static members. Instances are heap-allocated with `new`.

## Syntax

```dafny
class Point {
  var x: real
  var y: real

  method Dist2(that: Point) returns (z: real) {
    z := Norm2(x - that.x, y - that.y);
  }
}

// Named and anonymous constructors
class C {
  var x: int

  constructor(n: int) {        // anonymous
    x := n;
  }

  constructor Zero() {         // named
    x := 0;
  }
}

// Static members
class Counter {
  static const Max: nat := 100

  static method Create() returns (c: Counter) {
    c := new Counter;
  }
}

method Usage() {
  var c := new C(5);          // anonymous constructor
  var d := new C.Zero();      // named constructor
  var o: object := new Counter;   // object is the root supertype
}
```

## Notes

- `new C(args)` allocates on the heap and invokes the anonymous constructor; `new C.Name(args)` invokes a named constructor.
- A constructor assigns the object's fields; every allocation must satisfy the class invariant.
- `static` members belong to the class, not an instance, and are referenced as `C.Max` / `C.Create()`.
- `object` is the root supertype of all reference types — any class instance can be held in an `object` variable.
- Instance methods, constructors, and static method calls all bind results to variables like any method: `var c := Counter.Create();`.

## Two-phase constructors

Constructor body has two phases separated by `new;`: initialization phase (before) and post-initialization phase (after). In the init phase, `this` can only be used to assign fields. After `new;`, no restrictions. A `const` field without RHS may only be assigned in the init phase.

```dafny
class TwoPhase {
  const c: int   // must be set in init phase if no default
  var x: int

  constructor(initVal: int)
    ensures fresh(this)
  {
    this.c := initVal;     // init phase: only field assignment allowed
    new;                    // separator
    this.x := initVal * 2; // post-init phase: full access to this
  }
}
```

See also dafny-frames for `fresh(this)` in constructor postconditions.
