---
name: dafny-triggers
description: Use when a Dafny quantifier proof fails, is flaky, or times out, or when Dafny warns "No terms found to trigger on" (a quantifier with no trigger) — controlling SMT instantiation with matching patterns and the {:trigger} attribute.
tags:
  - dafny
  - specification
  - proof
  - performance
related_skills:
  - dafny-quantifiers
  - dafny-logic
  - dafny-opaque-reveal
  - dafny-assert-assume
---

# Dafny: Quantifier Triggers

A **trigger** (matching pattern) tells the SMT solver *when* to instantiate a
`forall`/`exists`. The solver only instantiates a quantifier when a term in the
proof context syntactically matches one of its trigger patterns. Dafny picks
triggers automatically from the body, but the choice drives whether a proof
succeeds, fails, or loops — so it is worth overriding with `{:trigger ...}`.

## Syntax

```dafny
predicate P(x: int)
function f(x: int): int
function g(x: int): int

// Explicit trigger: instantiate this fact only when the term P(x)
// already appears as a ground term in the context.
lemma L() ensures forall x {:trigger P(x)} :: P(x) ==> P(x + 1)

// Multiple triggers — either pattern fires the quantifier.
ghost const c := forall x {:trigger f(x)} {:trigger g(x)} :: f(x) == g(x)
```

The `{:trigger e}` attribute goes right after the bound variable list, before
`::`. Provide several `{:trigger ...}` attributes to allow multiple patterns.

## What can (and can't) be a trigger

Only **uninterpreted** terms qualify: function/predicate applications, datatype
constructors/destructors, sequence/map/array selects (`s[i]`, `m[k]`). Arithmetic
(`+`, `<`), equality, and boolean operators **cannot** be triggers.

```dafny
// No valid trigger: the body is pure arithmetic, so Dafny warns
// "quantifier has no trigger" and instantiation is unreliable.
ghost const bad := forall x: int :: x + 1 > x;
```

Fix by wrapping the key term in a helper so there is a stable pattern:

```dafny
function Succ(x: int): int { x + 1 }
ghost const ok := forall x: int {:trigger Succ(x)} :: Succ(x) > x;
```

## Failure modes

- **Matching loop → timeout.** A trigger whose instantiation produces a new term
  that matches the same trigger loops forever:

  ```dafny
  // instantiating at f(x) yields f(f(x)), which matches f(_) again → loops.
  forall x {:trigger f(x)} :: f(x) == f(f(x))   // BAD
  ```

- **Too restrictive → never fires.** If no ground term matches the trigger, a
  true fact is never instantiated and the proof fails for no obvious reason.
- **No trigger.** Arithmetic-only bodies (above) have none; the solver may still
  guess, but proofs become flaky across versions/seeds.

## Notes

- A trigger must mention **every** quantified variable, or Dafny rejects it.
- Symptom → cause: unexplained quantifier proof failure ≈ too-tight/missing
  trigger; sudden timeout after adding a quantifier ≈ matching loop.
- Give the solver stable patterns by phrasing bodies through predicates/functions
  (see `dafny-quantifiers`); use `opaque`/`reveal` (see `dafny-opaque-reveal`) to
  stop definitions from unfolding and flooding the context with terms.
