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

## Function fields

A datatype field can hold a first-class function value, enabling data-driven computation patterns like state machines with transition functions.

```dafny
// `!S` / `!A` mark the parameters non-variant, required because they appear
// to the left of an arrow in the `transition` field.
datatype DFA<!S(==), !A(==)> = DFA(
    states: set<S>,
    transition: (S, A) -> S,   // function value as a field
    startState: S,
    acceptStates: set<S>
)

function Apply<S(==), A(==)>(d: DFA<S, A>, s: S, a: A): S {
  d.transition(s, a)
}
```

## Notes

- Constructors may carry named fields (e.g. `head`, `tail`) accessible as destructors.
- Deconstruct values with `match` over each constructor case.
- `datatype` is inductive: every value is finite and well-founded.
- `codatatype` is coinductive: it admits infinite values like endless streams.
- A field may be a function type (e.g. `(S, A) -> S`), storing a first-class transition or computation as data.
- A type parameter appearing left of an arrow must be declared non-variant (`!S`) or contravariant (`-S`), otherwise Dafny reports *"formal type parameter 'S' is not used according to its variance specification"*.
