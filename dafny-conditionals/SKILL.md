---
name: dafny-conditionals
description: Use when writing or reading Dafny conditionals (`if/else` statement, `if/then/else` expression) — branch control flow versus producing a value.
tags:
  - dafny
  - statement
  - expression
  - control-flow
related_skills:
  - dafny-match
  - dafny-loops
  - dafny-such-that-and-binding-guard
---

# Dafny: Conditionals

The statement form `if C { ... } else { ... }` chooses which code runs. The expression form `if C then a else b` yields a value.

## Syntax

```dafny
method Example(x: int, y: int) returns (m: int)
{
  // Statement form: braces are MANDATORY
  if x < y {
    m := x + 1;
  } else {
    m := y - 1;
  }

  // Expression form: no braces, evaluates to a value
  m := if x < y then x else y;   // minimum of x and y
}
```

## Notes

- Statement `if` requires braces `{ }` on both branches; omitting them is a syntax error (unlike C).
- Expression `if C then a else b` has no braces and must always include the `else`.
- The `else` of a statement `if` is optional; the expression form's `else` is required.
- Use the expression form inside declarations, `return`, and specs; use the statement form for method bodies with side effects.
