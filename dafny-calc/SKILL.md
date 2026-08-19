---
name: dafny-calc
description: Use when writing or reading Dafny calculational proofs (calc) — a structured chain of steps joined by relational operators that proves an (in)equality transitively. Also triggers on the parser/verifier errors "the main operator of a calculation must be transitive" and "this operator cannot continue this calculation".
tags:
  - dafny
  - specification
  - proof
related_skills:
  - dafny-lemma
  - dafny-assert-assume
  - dafny-logic
---

# Dafny: calc

`calc` builds an equational/relational proof as a sequence of expressions linked by relations that compose transitively.

## Syntax

```dafny
lemma Assoc(x: int, y: int, z: int)
  ensures x + (y + z) == (x + y) + z
{
  calc {
    x + (y + z);
    == (x + y) + z;
  }
}
```

## Notes

- Each step is an expression terminated by `;`; the relation before a step (`==`, `<=`, `<`, `==>`, ...) links it to the previous one.
- The chain composes transitively, so the first and last expressions are related by the combined operator.
- A default relation can be set once with `calc == { ... }`; individual steps may override it.
- Hints (asserts, lemma calls) can be placed between steps inside `{ ... }` to justify a link.
