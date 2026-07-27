---
name: dafny-coinduction
description: Use when writing or reading Dafny coinductive constructs (codatatype, greatest predicate/lemma) — reasoning about infinite structures and co-inductive properties.
tags:
  - dafny
  - declaration
  - proof
related_skills:
  - dafny-datatype
  - dafny-ordinals
---

# Dafny: Coinduction

Coinduction enables reasoning about potentially infinite data structures (codatatypes) and co-inductive properties (greatest predicates/lemmas).

## Codatatypes

```dafny
// Infinite stream of integers
codatatype IStream<T> = ICons(head: T, tail: IStream<T>)

function Mult(a: IStream<int>, b: IStream<int>): IStream<int>
{
  ICons(a.head * b.head, Mult(a.tail, b.tail))
}
```

## Greatest predicates (co-inductive properties)

A `greatest predicate` defines the largest relation satisfying its body — used for co-inductive properties like "stream a is elementwise below stream b".

```dafny
// Co-inductive property: every element of a <= corresponding element of b
greatest predicate Below(a: IStream<int>, b: IStream<int>)
{
  a.head <= b.head && (a.head == b.head ==> Below(a.tail, b.tail))
}
```

## Greatest lemmas (co-inductive proofs)

A `greatest lemma` proves a property by coinduction — the recursive call serves as the coinduction hypothesis.

```dafny
// Prove: every element of a <= corresponding element of a*a
greatest lemma Theorem_BelowSquare(a: IStream<int>)
  ensures Below(a, Mult(a, a))
{
  assert a.head <= Mult(a, a).head;
  if a.head == Mult(a, a).head {
    Theorem_BelowSquare(a.tail);  // coinduction hypothesis on tail
  }
}
```

## Prefix predicates and lemmas

For finite approximations of infinite structures, use `#` notation:

- `Pos#[k]` — predicate holds for first k elements
- `==#[k]` — two codatatypes are equal up to depth k

```dafny
// Prove equality up to depth 10
assert a ==#[10] b;  // first 10 heads and tails match
```

## Notes

- `codatatype` admits infinite values (e.g., endless streams); `datatype` is strictly finite/inductive.
- Co-recursive functions on codatatypes need no decreases clause — they produce potentially infinite structure.
- `greatest predicate` defines the largest fixed point of its body; `least predicate` defines the smallest. Neither is the default — a plain `predicate` is an ordinary recursive definition and is not interchangeable with either (Dafny cannot prove a plain `predicate` equal to the `least predicate` with the same body).
- `greatest lemma` proves by coinduction: recursive calls are treated as coinduction hypotheses, not requiring a decreasing metric.
