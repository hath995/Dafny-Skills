---
name: dafny-opaque-reveal
description: Use when writing or reading Dafny hidden function bodies (opaque, reveal) — hides a function body from the verifier and exposes it selectively per scope.
tags:
  - dafny
  - declaration
  - proof
related_skills:
  - dafny-function-and-predicate
  - dafny-lemma
---

# Dafny: Opaque and Reveal

`opaque` hides a function's body from the verifier; `reveal` makes it visible in a chosen scope.

## Syntax

```dafny
opaque function Square(x: int): int
{
  x * x
}

lemma UseIt()
  ensures Square(3) == 9
{
  reveal Square(); // body visible only inside this lemma
}
```

## Notes

- `opaque` stops the verifier from unfolding the body, which can speed up proofs.
- Without a `reveal`, callers only know the function's signature, not its definition.
- `reveal Square();` exposes the body in the current scope only, not globally.
- Use opaque bodies to keep large proofs tractable, revealing on demand.
