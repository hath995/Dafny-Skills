---
name: dafny-quantifiers
description: "Use when writing or reading Dafny quantifier expressions (forall vars :: body, exists vars :: body, with `| P` filters and `<-` collection domains) — boolean expressions asserting a body holds for all or some values."
tags:
  - dafny
  - specification
  - expression
  - proof
related_skills:
  - dafny-forall-statement
  - dafny-triggers
  - dafny-logic
  - dafny-assert-assume
  - dafny-set
---

# Dafny: Quantifier Expressions

A quantifier expression is a `bool` expression: `forall` says the body (after
`::`) holds for **every** combination of the quantified variables in their
domain; `exists` says it holds for **at least one**.

## Syntax

```dafny
forall x: int :: x > 0                       // (false) claims body holds for every x
forall x: nat | x < 10 :: x * x < 100        // domain restricted by `| P`
exists x: int :: x * x == 25                 // (true) some x works

// In assertions / specs:
assert forall x: nat | x <= 5 :: x * x <= 25;
assert exists d :: 100 < d < 200;

// Nested, with implication (a very common shape):
assert forall n :: 2 <= n ==> (exists d :: n < d < 2*n);

// Multiple domains, drawing from a collection with `<-`:
// for each index x of s, every element y of s[x] is < x
assert forall x: nat | 0 <= x < |s|, y <- s[x] :: y < x;
```

## Quantifier domain

Everything between the variable and `::` is the *domain*. Each variable can carry:

- a type (`x: nat`) — optional; Dafny infers it from the body, and it is an
  error if it cannot,
- a `<- C` collection domain — ranges over the elements of `C`,
- a `| P` filter — restricts to values where `P` holds.

`forall x: T | P :: Q` is equivalent to `forall x: T :: P ==> Q`;
`exists x: T | P :: Q` is equivalent to `exists x: T :: P && Q`. The quantified
identifiers are bound only within the quantifier expression.

## Notes

- This is an **expression** (evaluates to `bool`) — used in `assert` / `assume` /
  `requires` / `ensures` / `invariant` / function bodies. It is distinct from the
  `forall` **statement** (a proof/aggregate command); see the
  `dafny-forall-statement` skill for that form.
- Idiom: a `forall` body usually uses `==>` (`forall x :: P(x) ==> Q(x)`); an
  `exists` body usually uses `&&` (`exists x :: P(x) && Q(x)`).
- The body follows `::`; multiple variables and their domains are comma-separated.
- Keep collection domains bounded (`| 0 <= i < |s|`) so the range is decidable.
- See `dafny-logic` for the boolean connectives that combine inside the body.

## Triggers

How the SMT solver decides *when* to instantiate a quantifier is governed by its
**triggers** (matching patterns). This is the usual reason a quantified fact
fails to prove, proves flakily, or times out — and Dafny may warn that a
quantifier has "no trigger". When that happens, reach for the `dafny-triggers`
skill, which covers the `{:trigger ...}` attribute and matching-loop pitfalls.

```dafny
// Override the auto-selected pattern: only fire when P(x) is a ground term.
forall x {:trigger P(x)} :: P(x) ==> P(x + 1)
```
