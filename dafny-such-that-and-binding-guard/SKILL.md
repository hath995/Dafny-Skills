---
name: dafny-such-that-and-binding-guard
description: Use when writing or reading Dafny such-that assignment and binding guards (`:|`, `:| assume`, `if x :| P`) — pick an arbitrary value satisfying a predicate. Also triggers on the compiler error "Dafny's heuristics cannot find any bound for variable" when `:|` can't be compiled.
tags:
  - dafny
  - statement
  - control-flow
related_skills:
  - dafny-var-and-assignment
  - dafny-conditionals
  - dafny-logic
---

# Dafny: Such-That Assignment and Binding Guards

`:|` assigns some value that satisfies a predicate ("choose any x such that P"). As a guard, `if x :| P` binds a witness when one exists, else takes the else branch.

## Syntax

```dafny
method Example(Y: set<int>, s: set<int>)
{
  // Such-that assignment: pick some y in Y (Dafny must prove one exists)
  var y :| y in Y;

  // Pick any y in the open interval (0, 10)
  y :| y > 0 && y < 10;

  // 'assume' skips the existence proof Dafny would otherwise demand
  y :| assume y in Y;

  // Binding guard: bind a witness if one exists, else run else branch
  if x :| x in s && x > 0 {
    print x, "\n";
  } else {
    // no such x exists in s
  }
}
```

## Notes

- Plain `y :| P` obliges Dafny to prove some value satisfying `P` exists; failure is a verification error.
- `y :| assume P` tells Dafny to trust existence without proof — convenient but unsound if wrong.
- The chosen value is arbitrary among those satisfying `P`; do not assume determinism.
- A binding guard `if x :| P { ... } else { ... }` introduces `x` scoped to the then-branch.
