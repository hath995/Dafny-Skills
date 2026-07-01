---
name: dafny-print-and-expect
description: Use when writing or reading Dafny runtime output and checks (`print`, `expect`) — emit values at runtime and assert conditions dynamically.
tags:
  - dafny
  - statement
  - io
related_skills:
  - dafny-assert-assume
  - dafny-method
---

# Dafny: Print and Expect

`print` writes comma-separated values to stdout at runtime. `expect` is a runtime assertion that throws if false — no static proof required.

## Syntax

```dafny
method Example(x: int)
{
  // print: comma-separated values, no automatic spaces or newline
  print "x = ", x, "\n";

  // expect: checked at runtime; raises an exception if the condition is false
  expect x > 0, "must be positive";
}
```

## Notes

- `print` takes a comma-separated list of values; add `"\n"` explicitly for a newline.
- `expect cond, msg;` is verified at *runtime* — if `cond` is false the program halts with `msg`; no compile-time proof is demanded.
- Contrast with `assert cond;`, which is *statically* verified by Dafny and imposes no runtime cost.
- Use `expect` in tests and `Main`; use `assert` to guide the verifier in proofs.
