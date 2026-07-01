---
name: dafny-forall-statement
description: Use when writing or reading the Dafny `forall` STATEMENT (proof/aggregate form, `forall i | P { ... }`, `forall ... ensures ... { }`) — not the quantifier expression.
tags:
  - dafny
  - statement
  - proof
  - control-flow
related_skills:
  - dafny-quantifiers
  - dafny-lemma
  - dafny-logic
  - dafny-array
---

# Dafny: Forall Statement

The `forall` statement is the proof/aggregate form (distinct from the `forall` quantifier in expressions). It does a bulk assignment, invokes a lemma over a range, or proves a universally quantified fact.

## Syntax

```dafny
// (a) Aggregate assignment: all writes happen simultaneously
method Copy(a: array<int>, b: array<int>, m: int, n: int)
  modifies b
{
  forall i | 0 <= i < n {
    b[i] := a[m + i];
  }
}

// (b) Lemma call over a range: if Lemma ensures Q(x), this establishes
//     forall x :: P(x) ==> Q(x)
method ProveRange(n: int)
{
  forall x | 0 <= x < n {
    Lemma(x);
  }
}

// (c) Proof of a quantified fact with an explicit body
lemma AgreeOnDomain(dom: set<int>)
  ensures forall p :: p in dom ==> f(p) == g(p)
{
  forall p | p in dom ensures f(p) == g(p) {
    // proof body establishing f(p) == g(p) for this p
  }
}
```

## Notes

- Aggregate form: every `b[i]` write uses the pre-state values and all occur at once; the assigned l-values must be distinct.
- Lemma-call form: calling `Lemma(x)` for each `x` in the range makes its `ensures` available as a universally quantified fact afterward.
- Proof form uses `ensures Q` plus a body proving `Q` for the bound variable; it discharges a `forall :: ... ==> Q` obligation.
- This is a statement (appears in bodies); the `forall x :: ...` quantifier is an expression used in specs — see the `dafny-quantifiers` skill.
