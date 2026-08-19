---
name: dafny-ghost-and-const
description: Use when writing or reading Dafny specification-only or immutable state (ghost, const, ghost const, static const) — marks values erased at compile time or fixed constants. Also triggers when Dafny reports a ghost variable, or a call to a ghost function/method, "allowed only in specification contexts".
tags:
  - dafny
  - declaration
  - ghost
related_skills:
  - dafny-lemma
  - dafny-function-and-predicate
---

# Dafny: Ghost and Const

`ghost` marks specification-only state erased at compile time; `const` declares an immutable named value.

## Syntax

```dafny
class Logger {
  ghost var history: seq<int>

  ghost method Log(v: int)
    modifies this
  {
    history := history + [v];
  }
}

const MAX: nat := 100
static const VERSION: string := "1.0"
ghost const LIMIT: int := 50
```

## Notes

- Ghost state exists only for specification/verification; it cannot influence compiled, executable code.
- Ghost declarations (vars, methods, functions) are erased at compile time.
- `const` is an immutable named constant, assigned once at declaration.
- `const` may be `static` (shared per type) and may also be `ghost`.
