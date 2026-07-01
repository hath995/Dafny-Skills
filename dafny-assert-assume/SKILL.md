---
name: dafny-assert-assume
description: Use when writing or reading Dafny proof statements (assert, assume) — assert is statically checked, assume is trusted without proof.
tags:
  - dafny
  - specification
  - proof
related_skills:
  - dafny-calc
  - dafny-logic
  - dafny-print-and-expect
---

# Dafny: assert / assume

`assert` states a fact the verifier must prove; `assume` states a fact the verifier accepts on trust.

## Syntax

```dafny
method Checked(x: int)
  requires x > 0
{
  assert x > 0;        // proven from the precondition
  assert x * x > 0;    // proven; also guides the verifier
}

method Trusted(x: int)
{
  assume x > 1;                 // accepted WITHOUT proof
  assert 2 * x + x / x > 3;     // now provable given the assumption
}
```

## Notes

- `assert` is verified: if Dafny cannot prove it, verification fails. It also acts as a hint that helps prove later goals.
- `assume` is trusted: taken as true with no proof. It is unsound and can hide bugs.
- Use `assume` only as scaffolding during development; discharge or remove it before shipping.
