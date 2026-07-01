---
name: dafny-match
description: Use when writing or reading Dafny pattern matching (`match`, `case`, constructor patterns like `Blue(n)`, wildcard `_`) — dispatch on a datatype value.
tags:
  - dafny
  - statement
  - expression
  - control-flow
related_skills:
  - dafny-datatype
  - dafny-conditionals
---

# Dafny: Match

`match` dispatches on the constructor of a datatype value; each `case` matches one constructor and can bind its fields.

## Syntax

```dafny
datatype Color = Red | Blue(n: int) | Green

method Describe(c: Color)
{
  match c {
    case Red      => print "red\n";
    case Blue(n)  => print n, "\n";   // n binds Blue's field
    case _        => print "other\n"; // wildcard catches the rest
  }
}
```

## Notes

- `case Blue(n) =>` binds the constructor's field(s) to fresh names usable in that arm.
- `case _ =>` is the wildcard, matching any remaining constructor.
- Cases must be exhaustive; Dafny errors if some constructor is unhandled and there is no `_`.
- `match` also works as an expression: `var s := match c { case Red => "r" case _ => "x" };`.
