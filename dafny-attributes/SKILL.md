---
name: dafny-attributes
description: Use when writing or reading Dafny declaration attributes ({:attr}, @Attr syntax) — metadata controlling verification, compilation, and tool behavior.
tags:
  - dafny
  - declaration
related_skills:
  - dafny-check-testing
---

# Dafny: Attributes

Attributes annotate declarations to control verification, compilation, and tooling. Older attributes use the `{:attr args}` form; newer Dafny versions add an `@Attr(args)` form, but the two sets are **not** in one-to-one correspondence — some attributes exist in only one form.

## Syntax

```dafny
// @-syntax:
@IsolateAssertions
method SplitAssertions() { }

@Fuel(2, 3)
lemma HighFuelProof() { }

@TimeLimit(20)
method TimeoutLimited() { }

// Traditional {:attr} syntax:
method {:verify false} SkipVerification() { }
method {:fuel 2, 3} OldStyleFuel() { }
method {:extern "native_impl"} ExternalImpl()
```

## Common attributes on methods/functions

| Attribute | Effect |
|-----------|--------|
| `{:test}` / `@Test` | Marks method as a test entry point for `dafny test` |
| `{:verify false}` | Skips verification entirely (no `@` equivalent — `@Verify` takes no arguments and does **not** disable verification) |
| `{:axiom}` / `@Axiom` | Marks a bodyless declaration as trusted, no proof obligation |
| `{:fuel N, M}` / `@Fuel(N, M)` | Sets how many times a function definition may be unfolded during proof |
| `{:timeLimit N}` / `@TimeLimit(N)` | Per-declaration prover timeout in **seconds** (CLI default is 30) |
| `{:extern "name"}` | Provides external implementation for compilation (no `@Extern` in Dafny 4.10) |
| `{:isolate_assertions}` / `@IsolateAssertions` | Verifies each assertion in its own batch |
| `{:concurrent}` / `@Concurrent` | Asserts the declaration is safe to call concurrently — requires provably empty `reads`/`modifies` frames |

## Notes

- Where both forms exist, `@AttrName(args)` corresponds to the traditional `{:attr_name args}` form — but check that the `@` form exists before using it. In Dafny 4.10 `@Extern`, `@Induction`, and `@AssumeConcurrent` do not resolve.
- Attribute names are **case-sensitive**: `{:verify false}` disables verification, while `{:Verify false}` is silently ignored as an unknown attribute and the body is still verified.
- Attribute parameters are comma-separated. `{:fuel 2 3}` is a parse error; write `{:fuel 2, 3}`.
- `{:verify false}` emits a warning recommending a bodyless declaration with `{:axiom}` instead — under `--allow-warnings:false` that warning fails the build.
- `{:concurrent}` is a concurrency-safety obligation, not a switch for parallel verification. Use `--cores` / `--isolate-assertions` for verification throughput.
- See dafny-check-testing for `{:test}` usage with DafnyCheck property-based testing.
