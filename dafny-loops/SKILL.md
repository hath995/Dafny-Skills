---
name: dafny-loops
description: Use when writing or reading Dafny loops (`while`, `for ... to`/`downto`, `break`, `continue`, `label`) — iteration and loop control flow.
tags:
  - dafny
  - statement
  - control-flow
related_skills:
  - dafny-invariant
  - dafny-decreases
  - dafny-forall-statement
---

# Dafny: Loops

`while` and `for` iterate; `break`/`continue` alter flow; a `label` lets `break`/`continue` target an outer loop.

## Syntax

```dafny
method Example(a: array<int>, n: int)
{
  var x, y := 20, 3;

  // while: braces mandatory
  while x > y {
    x := x - y;
  }

  // for over a range: i takes 0,1,...,9
  for i := 0 to 10 {
    print i, "\n";
  }

  // downto counts backwards: i takes 9,8,...,0
  for i := 10 downto 0 {
    print i, "\n";
  }

  // labeled loop targeted by break
  label L: while true {
    if x == 0 { break L; }
    x := x - 1;
  }

  // continue skips to the next iteration
  var i := 0;
  while i < n {
    if a[i] == 0 { i := i + 1; continue; }
    i := i + 1;
  }
}
```

## Notes

- Braces `{ }` are mandatory on `while` and `for` bodies.
- `for i := lo to hi` runs `i = lo..hi-1`; `for i := hi downto lo` runs `i = hi-1..lo` (bounds are exclusive on the moving end).
- `break L;` / `continue L;` refer to a `label L:` on an enclosing loop; bare `break`/`continue` act on the innermost loop.
- To verify, `while` loops usually need an `invariant` and a `decreases` clause — see the `dafny-invariant` and `dafny-decreases` skills.
