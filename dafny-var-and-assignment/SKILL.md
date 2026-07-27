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

## Failure-aware assignment (`:-`)

The `:-` operator unwraps values from a *failure-compatible* type — one declaring `IsFailure()`, `PropagateFailure()`, and `Extract()`. If successful, binds the inner value; if failed, the method returns immediately with the propagated error. Works like monadic bind for early-exit patterns.

```dafny
datatype Err = Neg

datatype Result<T> = Success(value: T) | Failure(error: Err) {
  predicate IsFailure() { this.Failure? }
  function PropagateFailure<U>(): Result<U> requires IsFailure() { Failure(this.error) }
  function Extract(): T requires !IsFailure() { this.value }
}

method Test(x: int) returns (res: Result<bool>) {
  if x < 0 { return Failure(Neg); }
  return Success(true);
}

method Caller() returns (res2: Result<bool>) {
  var unwrapped :- Test(6);    // on failure, returns PropagateFailure() immediately
  // here, unwrapped is just bool (unwrapped via Extract())
  return Success(unwrapped);   // if we get here, Test succeeded
}
```

## Notes

- In `a, b := e1, e2` every right-hand side is evaluated using the *old* values before any target is written, so `x, y := y, x` swaps without a temp.
- Left-hand targets in a parallel assignment must be distinct l-values.
- A bare `var n: int;` is declared but unassigned; reading it before assignment is a verification error.
- `var m := 5;` infers the type from the initializer; add `: T` when you want to pin the type.
- `var v :- expr` requires the RHS type to declare exactly `IsFailure()`, `PropagateFailure()`, and `Extract()`. Any other naming (e.g. `IsSuccess`/`Value`/`Error`) fails with *"member IsFailure does not exist ... in :- statement"*.
- On success `:-` binds `Extract()`; on failure it returns `PropagateFailure()` immediately from the enclosing method, which must therefore return a compatible error-wrapped type.
