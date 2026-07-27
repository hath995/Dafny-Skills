---
name: dafny-ordinals
description: Use when writing or reading Dafny ordinal types (ORDINAL, ω) — values larger than any nat, used for transfinite termination metrics and coinductive reasoning.
tags:
  - dafny
  - type
related_skills:
  - dafny-primitive-types
  - dafny-decreases
---

# Dafny: Ordinal Type

`ORDINAL` is a well-founded type whose values extend beyond the naturals: every `nat` is an ordinal, and non-nat ordinals also exist (`o.IsNat` is not provable for an arbitrary `o: ORDINAL`). The smallest non-nat ordinal is ω. Used mainly for transfinite termination metrics and as the index type of prefix predicates in coinductive proofs.

## Syntax

```dafny
// Any nat literal converts to a finite ordinal. There is no literal for ω.
ghost function Finite(n: nat): ORDINAL { n as ORDINAL }

// Recurse down a successor ordinal. Subtraction needs a positive finite offset.
lemma DownToLimit(o: ORDINAL)
  decreases o
{
  if o.Offset > 0 {
    DownToLimit(o - 1);
  }
}
```

## Properties

- `o.IsLimit` — true for limit ordinals (0, ω, ω+ω, ...); these have `o.Offset == 0`
- `o.IsSucc` — true for successor ordinals (n+1, ω+1, ...)
- `o.IsNat` — true if the ordinal is a natural number
- `o.Offset` — the finite offset from the largest limit ordinal below `o`

## Operations

- Comparison: `==`, `<`, `<=`, etc. (total well-founded order); `o + 1 > o` for every `o`
- Addition: associative but **not** commutative — `1 + ω == ω`, while `ω + 1 > ω`
- Subtraction: only when the RHS is a `nat` and `o.Offset` is at least that large. `o - 1` on a limit ordinal is a verification error (`ORDINAL subtraction might underflow a limit ordinal`), so guard with `o.Offset > 0`

## Conversion

```dafny
method Convert(someOrd: ORDINAL) {
  var n: nat := 5;
  var o: ORDINAL := n as ORDINAL;    // nat -> ORDINAL always valid
  if someOrd.IsNat {
    var m: nat := someOrd as nat;    // ORDINAL -> nat, only when .IsNat holds
  }
}
```

## Notes

- Chiefly a specification type for termination metrics, but not ghost-only: as of Dafny 4.10 `ORDINAL` locals, arithmetic, and datatype fields do compile and run.
- Common in coinductive termination metrics where a simple `nat` decreases clause is insufficient — see the `P#[k]` prefix-predicate notation in dafny-coinduction.
- There is no way to write ω directly. `n as ORDINAL` always yields the *finite* ordinal `n`: `(1 as ORDINAL).IsNat` holds and `(1 as ORDINAL) == 1`. Non-nat ordinals arise from the logic (e.g. as prefix-predicate indices), not from literals.
