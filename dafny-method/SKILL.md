---
name: dafny-method
description: Use when writing or reading Dafny methods (method, returns, return) — imperative routines with named out-parameters that callers must bind to variables.
tags:
  - dafny
  - declaration
  - oo
related_skills:
  - dafny-requires-ensures
  - dafny-class-and-constructor
  - dafny-var-and-assignment
---

# Dafny: Method

A method is an imperative routine. Outputs are declared with `returns` as named out-parameters that are assigned in the body.

## Syntax

```dafny
method Hello() {
  print "Hello\n";
}

method Norm2(x: real, y: real) returns (z: real) {
  z := x * x + y * y;
}

// Multiple return values
method Prod(x: int) returns (dbl: int, trpl: int) {
  dbl, trpl := x * 2, x * 3;
}

// Explicit / early return
method Abs(x: int) returns (r: int) {
  if x >= 0 {
    return x;
  }
  return -x;
}

method Caller() {
  var d, t := Prod(5);   // must bind results
  var a := Abs(-3);
}
```

## Notes

- Named return variables are assigned in the body; no `return` statement is needed unless you want an early exit.
- `return e1, e2;` is shorthand for assigning the out-parameters then exiting.
- Callers MUST bind every result to a variable: `var d, t := Prod(5);`. A method call is a statement, not an expression — it cannot appear inside a larger expression.
- Multiple outputs are assigned together: `dbl, trpl := x*2, x*3;`.
