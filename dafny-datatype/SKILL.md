---
name: dafny-datatype
description: Use when writing or reading Dafny algebraic types (datatype, codatatype, constructors, match) — inductive/finite and coinductive/infinite tagged-union types.
tags:
  - dafny
  - declaration
  - type
related_skills:
  - dafny-match
  - dafny-newtype-and-subset
  - dafny-tuple
---

# Dafny: Datatype and Codatatype

`datatype` declares inductive (finite) algebraic types; `codatatype` declares coinductive (possibly infinite) ones.

## Syntax

```dafny
datatype List<T> = Nil | Cons(head: T, tail: List<T>)

datatype Option<T> = Some(value: T) | None

codatatype Stream<T> = Cons(head: T, tail: Stream<T>)

function Length<T>(l: List<T>): nat
{
  match l
  case Nil => 0
  case Cons(_, tail) => 1 + Length(tail)
}
```

## Notes

- Constructors may carry named fields (e.g. `head`, `tail`) accessible as destructors.
- Deconstruct values with `match` over each constructor case.
- `datatype` is inductive: every value is finite and well-founded.
- `codatatype` is coinductive: it admits infinite values like endless streams.
