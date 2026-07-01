---
name: dafny-var-and-assignment
description: Use when writing or reading Dafny variable declarations and assignment (`var`, `:=`, parallel/simultaneous assignment `x, y := a, b`) — how to declare, initialize, and swap.
tags:
  - dafny
  - statement
  - control-flow
related_skills:
  - dafny-such-that-and-binding-guard
  - dafny-method
---

# Dafny: Variables and Assignment

`var` declares a local variable; `:=` assigns to it. Multiple targets on the left of one `:=` are assigned simultaneously.

## Syntax

```dafny
method Example()
{
  var nish: int;                  // declared, uninitialized
  var m := 5;                     // type inferred as int
  var i: int, j: nat;             // several declarations at once
  var x, y, z: bool := 1, 2, true; // declare + initialize together

  z := false;                     // ordinary assignment

  // Parallel (simultaneous) assignment: RHS all evaluated first
  x, y := x + y, x - y;

  // Because RHS is evaluated before any store, this swaps cleanly:
  x, y := y, x;
}
```

## Notes

- In `a, b := e1, e2` every right-hand side is evaluated using the *old* values before any target is written, so `x, y := y, x` swaps without a temp.
- Left-hand targets in a parallel assignment must be distinct l-values.
- A bare `var n: int;` is declared but unassigned; reading it before assignment is a verification error.
- `var m := 5;` infers the type from the initializer; add `: T` when you want to pin the type.
