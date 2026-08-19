---
name: dafny-elephant-assignment
description: Use when writing or reading Dafny failure-propagating assignment (`:-`, the "elephant" operator, with `Outcome`/`Result`-style types) — bind on success, return on failure. Surfaces as the verifier error "does not have a member IsFailure" when the right-hand side isn't failure-compatible.
tags:
  - dafny
  - statement
  - control-flow
  - error-handling
related_skills:
  - dafny-method
  - dafny-datatype
---

# Dafny: Elephant Assignment

`:-` ("elephant") unwraps a failure-compatible value: on success it binds the inner value; on failure it makes the enclosing method return that failure immediately.

## Syntax

```dafny
datatype Outcome<T> = Success(value: T) | Failure(error: string)

method MightFail() returns (r: Outcome<int>)

method Get() returns (r: Outcome<int>)
{
  // If MightFail() returns Failure, Get returns that Failure here.
  // Otherwise x binds the unwrapped success value.
  var x :- MightFail();
  return Success(x + 1);
}
```

## Notes

- `var x :- E;` short-circuits: a failing `E` causes the current method to return the propagated failure, so code after it only runs on success.
- The RHS type must be failure-compatible (implement the `IsFailure`/`PropagateFailure`/`Extract` members, as `Result`/`Outcome`-style datatypes do).
- The enclosing method's out-parameter type must be able to carry the propagated failure.
- Use `x :- E;` (no `var`) to assign into existing variables, and `:-` with no LHS to propagate failure while discarding the success value.
