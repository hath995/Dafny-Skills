---
name: dafny-assert-assume
description: Use when writing or reading Dafny proof statements (assert, assume) — assert is statically checked, assume is trusted without proof. Surfaces as the verifier error "assertion might not hold" when a failed assert can't be proved.
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

## assert-by

Use `assert ... by { proof-body }` to provide an explicit proof sketch that helps the verifier discharge a quantified assertion. The body typically contains a matching `forall`/`exists` statement with `ensures`.

```dafny
lemma Example(xs: seq<int>)
  requires forall i :: 0 <= i < |xs| ==> xs[i] >= 1
  ensures forall i :: 0 <= i < |xs| ==> xs[i] * xs[i] >= xs[i]
{
  assert forall i :: 0 <= i < |xs| ==> xs[i] * xs[i] >= xs[i] by {
    forall i | 0 <= i < |xs| ensures xs[i] * xs[i] >= xs[i]
    {
      assert xs[i] >= 1;   // proof body for each i
    }
  }
}
```

## {:axiom} attribute

Attach `{:axiom}` to `assume` or `expect` to skip verification entirely. Useful for parsing assumptions where proof is impractical; the runtime still checks `expect`.

```dafny
method Parse(parts: seq<string>) returns (data: string) {
  assume {:axiom} |parts| == 2;    // skip verification entirely
  expect {:axiom} |parts| > 0;     // runtime check, no proof obligation
  data := parts[1];
}
```

## Notes

- `assert` is verified: if Dafny cannot prove it, verification fails. It also acts as a hint that helps prove later goals.
- `assume` is trusted: taken as true with no proof. It is unsound and can hide bugs.
- Use `assume` only as scaffolding during development; discharge or remove it before shipping.
- `assert ... by { ... }` provides an explicit proof body for quantified assertions, often using a matching `forall`/`exists` with `ensures`.
- `{:axiom}` on `assume`/`expect` skips the verification obligation entirely; use sparingly for unprovable runtime assumptions.
