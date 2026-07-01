---
name: dafny-lemma
description: Use when writing or reading Dafny proof methods (lemma, ensures, requires, decreases) — a ghost method that proves a postcondition and is erased at compile time.
tags:
  - dafny
  - declaration
  - proof
  - ghost
related_skills:
  - dafny-calc
  - dafny-forall-statement
  - dafny-decreases
  - dafny-twostate
---

# Dafny: Lemma

A `lemma` is a ghost proof method: it derives its `ensures` clause and has no runtime effect.

## Syntax

```dafny
lemma AddComm(x: nat, y: nat)
  ensures x + y == y + x
{
  // empty: the verifier proves this automatically
}

lemma SumBound(n: nat)
  requires n > 0
  ensures n * n >= n
  decreases n
{
  if n > 1 {
    SumBound(n - 1); // recursive call needs decreases
  }
}
```

## Notes

- Proves the `ensures` from the `requires`; the body is a proof, not executable code.
- Body may be empty when the fact is automatic; otherwise fill it with proof steps.
- Recursive lemmas (induction) require a `decreases` clause to show termination.
- Calling a lemma at a use site "unlocks" its `ensures` for the verifier there.
- Lemmas are ghost — erased at compile time, no runtime cost.
