---
name: dafny-check-testing
description: How to write Dafny functions with pre/postconditions and property-based tests using the DafnyCheck library in this project.
version: 1.0.0
---

# Dafny + DafnyCheck Property-Based Testing

## Project Setup

- **Location**: `deps/dafnycheck` (or similar lemmata-managed project)
- **Toolchain**: lemmata manages dependencies; Dafny 4.10+
- **Verify command**: `dafny.exe verify "<path>"` — use forward slashes in WSL bash

## DafnyCheck Library

Located at `deps/dafnycheck/src/`. Key files:
- `DafnyCheck.dfy` — test runner entry points
- `Arbitrary.dfy` — generators (Arbitrary<T>)
- `RunConfig.dfy`, `TestResult.dfy`, `Reporting.dfy`

### Includes and Imports

```dafny
include "../deps/dafnycheck/src/Arbitrary.dfy"
include "../deps/dafnycheck/src/DafnyCheck.dfy"

module MyModule {
  import opened DafnyCheck
  import opened Arbitraries
}
```

### Generators (Arbitrary static methods)

| Generator | Usage |
|-----------|-------|
| `Arbitrary.Strings(minLen, maxLen, ascii)` | String generator |
| `Arbitrary.Nats(bound)` | Natural numbers [0, bound) |
| `Arbitrary.Chars()` | Printable ASCII chars |
| `Arbitrary.Bools()` | Boolean values |
| `Arbitrary.Range(min, max)` | Integers [min, max) |
| `Arbitrary.Just(value)` | Constant generator |

### Test Entry Points

```dafny
// Predicate-based property tests
DafnyCheck.RunTest(pred: T -> bool, arb: Arbitrary<T>, name: string) returns (passed: bool)
DafnyCheck.RunTestWithExamples(pred, arb, name, examples: nat)
DafnyCheck.RunTestWithConfig(pred, arb, name, cfg: RunConfig<T>)

// Method-under-test tests (for heap-mutating methods)
DafnyCheck.RunMethodTest<Input, E>(arb, sut: MethodUnderTest<Input, E>, name)
```

## Lemmas for Recursive Functions

When a recursive function has properties that Dafny can't prove as postconditions (e.g., "all chars from index N to end are '0'"), use a separate lemma:

```dafny
function CountTrailingZeros(s: string): nat
  requires 0 < |s|
  ensures CountTrailingZeros(s) <= |s| - 1
{ ... }

lemma CountTrailingZerosProps(s: string)
  requires 0 < |s|
  requires exists i :: 0 <= i < |s| && s[i] != '0'
  ensures (forall j :: |s| - CountTrailingZeros(s) <= j < |s| ==> s[j] == '0')
  decreases |s|
{
  if s[|s| - 1] == '0' {
    CountTrailingZerosProps(s[..|s|-1]);
  }
}

function removeTrailingZeros(num: string): string
  requires 0 < |num|
  requires exists i :: 0 <= i < |num| && num[i] != '0'
{
  var k := CountTrailingZeros(num);
  CountTrailingZerosProps(num);  // Call lemma to unlock its ensures-clauses
  assert (forall j :: |num| - k <= j < |num| ==> num[j] == '0');
  num[..|num|-k]
}
```

**Key rules:**
- Lemmas need a `decreases` clause for recursive calls
- Call the lemma from within the function body before asserts that depend on it
- Keep function postconditions minimal — only what verifies without lemmas

## Test Input Guards

When functions have requires-clauses, test predicates using arbitrary strings MUST filter invalid inputs:

```dafny
function IsValidNum(s: string): bool
{
  |s| >= 1 &&
  (forall i :: 0 <= i < |s| ==> '0' <= s[i] <= '9') &&
  s[0] != '0'
}

method {:test} TestSomething()
{
  var arb := Arbitrary<string>.Strings(1, 20, true);
  var _ := DafnyCheck.RunTest(
    (s: string) => if IsValidNum(s) then removeTrailingZeros(s)[|removeTrailingZeros(s)|-1] != '0' else true,
    arb,
    "result has no trailing zeros"
  );
}
```

Without guards, DafnyCheck generates arbitrary ASCII chars that violate function preconditions, causing verification failures in the test itself.

### Writing Tests

Tests are methods with the `{:test}` attribute. **CRITICAL**: attribute goes AFTER `method` keyword.

```dafny
// WRONG — causes "rbrace expected" parse error
{:test}
method TestSomething() { ... }

// CORRECT
method {:test} TestSomething() {
  var arb := Arbitrary.Strings(1, 20, true);
  DafnyCheck.RunTest(
    (s: string) => /* property predicate */,
    arb,
    "description"
  );
}
```

### Writing Functions with Preconditions/Postconditions

```dafny
function myFunction(input: string): string
  requires 0 < |input|
  ensures 0 <= |myFunction(input)|
  ensures /* meaningful postcondition */
{
  // implementation
}
```

## Pitfalls

1. **`{:test}` attribute placement**: Must be `method {:test} Name()`, never on a separate line before `method`.
2. **LSP false positives**: DafnyCheck imports `Std.Wrappers` which LSP may not resolve but IS available at compile time. Ignore "module Std does not exist" errors from the editor — verify with `dafny.exe`.
3. **Path handling in WSL bash**: Use forward slashes for Windows paths: `"C:/Users/..."` not `"C:\Users\..."`.
4. **Tuple patterns in set comprehensions:** `{ k | (k, _) in r }` fails — Dafny parses `|` as bitwise OR. Use indexed access: `{ r[i].0 | i .. |r| }`.
5. **Set operators `\cap`, `\cup`:** Fail with parse errors ("invalid NameSegment", "rbrace expected"). Use `{ x | x in s1 && x in s2 }` for intersection; `s1 + s2` for union.
6. **Sequence concatenation:** Uses `+`, not `++`.
7. **Nested map comprehensions:** `{ { k -> v | ... } | ... }` triggers 'invalid NameSegment'. Extract inner computation to a helper function or use sequences instead of maps for record-like structures.

## Common Property Patterns

- **Idempotence**: `f(f(x)) == f(x)`
- **No trailing zeros**: `result[|result|-1] != '0' || result == "0"`
- **Preserves value**: `original == result + "0"*count`
- **Length non-increasing**: `|f(x)| <= |x|`
- **Identity on subset**: `if x ends with non-zero then f(x) == x else true`
