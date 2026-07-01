---
name: dafny-trait
description: Use when writing or reading Dafny interfaces or inheritance (trait, extends) — declares abstract members that classes must implement.
tags:
  - dafny
  - declaration
  - oo
  - type
related_skills:
  - dafny-class-and-constructor
  - dafny-module
---

# Dafny: Trait

A `trait` is an interface-like reference type; a `class` uses `extends` to implement its members.

## Syntax

```dafny
trait Shape {
  method Area() returns (a: real)
}

class Circle extends Shape {
  var r: real

  method Area() returns (a: real)
  {
    a := 3.14 * r * r;
  }
}
```

## Notes

- Traits declare abstract members (methods, functions, fields) that implementers must provide.
- A class `extends` one or more traits, supplying concrete bodies.
- Traits are reference types, so a `Shape` value may point to any subclass.
- Enables polymorphism: code against the trait, run against any implementing class.
