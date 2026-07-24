---
name: dafny-check-testing
description: Use when writing property-based tests with the DafnyCheck library — generators (incl. custom/recursive datatypes), testing pure functions and heap-mutating methods, and the seeded/random runner.
version: 1.1.0
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

Include paths depend on how DafnyCheck is vendored (e.g.
`lemmata_deps/dafnycheck/src/`). Import the modules you actually use:

```dafny
include "../src/DafnyCheck.dfy"
include "../src/Arbitrary.dfy"

module MyTests {
  import opened DafnyCheck
  import opened Arbitraries      // generators (note the plural module name)
  import opened RunConfigs       // RunConfig, DefaultConfig, Verbosity
  import opened Std.Wrappers     // Option / Result (Some/None/Success/Failure)
}
```

### Generators & combinators

All generators are **static methods** that return an out-param — call them as
statements (`var g := Arbitrary<T>.Name(args);`), never inline as expressions.

Primitives:

| Call | Yields |
|------|--------|
| `Arbitrary<bool>.Bools()` | booleans |
| `Arbitrary<int>.Range(min, max)` | ints in `[min, max)` |
| `Arbitrary<nat>.Nats(bound)` | nats in `[0, bound)` |
| `Arbitrary<char>.Chars()` | printable ASCII chars |
| `Arbitrary<real>.Reals()` | reals |
| `Arbitrary<string>.Strings(minLen, maxLen, ascii)` | strings |
| `Arbitrary<T>.Just(value)` | the constant `value` |
| `Arbitrary<T>.Of([a, b, c])` | one of the listed values |
| `Arbitrary<bv8>.BitVectors8()` | bitvectors (also 1/2/16/32/64/128/256) |

Collections (element generator + size bounds):

| Call | Yields |
|------|--------|
| `Arbitrary.Lists(elem, minSize, maxSize)` | `seq<S>` |
| `Arbitrary.Sets(elem, minSize, maxSize)` | `set<S>` |
| `Arbitrary.Multisets(elem, minSize, maxSize)` | `multiset<S>` |
| `Arbitrary.Maps(keyGen, valGen, minSize, maxSize)` | `map<K,V>` |
| `Arbitrary.Arrays(elem, minSize, maxSize)` | `array<S>` |
| `Arbitrary<(A,B)>.Tuple(a, b)` … `Tuple10(...)` | tuples (name the result type explicitly) |

Combinators:

| Call | Effect |
|------|--------|
| `g.Map((x: T) => f(x))` | transform each generated value |
| `g.FlatMap(f)` | bind: generate a `T`, then build an `Arbitrary<U>` from it |
| `Arbitrary<T>.Mix([g1, g2, ...])` | one-of: pick a branch at random (branches need disjoint reprs) |

### Test entry points

```dafny
// Pure predicate properties:
RunTest(pred, arb, name)                        // 100 runs (default)
RunTestWithExamples(pred, arb, name, examples)  // fixed run count
RunTestWithConfig(pred, arb, name, cfg)         // RunConfig: seed, verbosity, ...

// Heap-mutating "method under test":
RunMethodTest(arb, sut, name)
RunMethodTestWithExamples(arb, sut, name, examples)
RunMethodTestWithConfig(arb, sut, name, cfg)
```

### Testing a pure function

```dafny
method {:test} TestInRange() {
  var arb := Arbitrary<int>.Range(0, 100);
  var _ := RunTest((n: int) => 0 <= n < 100, arb, "in range");
}
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

## Generating custom datatypes

Build a generator for a `datatype` by generating its fields with primitive
generators, wrapping each constructor with `Map`, and combining variants with
`Mix`. **Each branch given to `Mix` must have a disjoint `repr`** — build a fresh
element generator per branch; never reuse one `Arbitrary` across two branches.

```dafny
datatype Shape = Circle(r: nat) | Rect(w: nat, h: nat)

method ShapeArb() returns (arb: Arbitrary<Shape>)
  ensures arb.Valid()
{
  var rGen := Arbitrary<nat>.Nats(100);
  var circ := rGen.Map((r: nat) => Circle(r));

  var wGen := Arbitrary<nat>.Nats(100);
  var hGen := Arbitrary<nat>.Nats(100);
  var dims := Arbitrary<(nat, nat)>.Tuple(wGen, hGen); // fresh gens => disjoint reprs
  var rect := dims.Map((wh: (nat, nat)) => Rect(wh.0, wh.1));

  arb := Arbitrary<Shape>.Mix([circ, rect]);
}
```

**Recursive datatypes** use a `Registry<T>` with `Tie` (lazy self-reference),
`Register`, and `Lookup`, bounded by a max depth:

```dafny
datatype Tree = Leaf(n: int) | Node(kids: seq<Tree>)

method BuildTree() returns (arb: Arbitrary<Tree>) {
  var reg := new Registry<Tree>("Leaf", 4);          // base variant, max depth

  var ints := Arbitrary<int>.Range(0, 100);
  var leafArb := ints.Map((n: int) => Leaf(n));
  reg.Register("Leaf", leafArb);

  var elem := reg.Tie("Tree");                        // lazy reference to "Tree"
  var kids := Arbitrary<Tree>.Lists(elem, 0, 3);
  var nodeArb := kids.Map((ks: seq<Tree>) => Node(ks));
  reg.Register("Node", nodeArb);

  var treeArb := Arbitrary<Tree>.Mix([reg.Tie("Leaf"), reg.Tie("Node")]);
  reg.Register("Tree", treeArb);

  arb := reg.Lookup("Tree");
}
```

## Testing methods (heap-mutating)

For methods rather than pure functions, implement the
`MethodUnderTest<Input, E>` trait and pass an instance to a `RunMethodTest*`
entry point. `run` returns a `Result`: `Success(true)` = property held,
`Success(false)` / `Failure(e)` = counterexample (with payload `e`).

```dafny
class DedupSut extends MethodUnderTest<seq<int>, string> {
  constructor() ensures fresh(this) ensures Valid() {}
  ghost predicate Valid() reads this { true }

  method run(input: seq<int>) returns (result: Result<bool, string>)
    requires Valid() ensures Valid() decreases 0
  {
    var out := Dedup(input);                 // the method under test
    if HasDuplicates(out) {
      result := Failure("output has duplicates");
    } else if SeqSet(out) != SeqSet(input) {
      result := Failure("output set != input set");
    } else {
      result := Success(true);
    }
  }
}

method {:test} TestDedup() {
  var elems := Arbitrary<int>.Range(0, 10);
  var arb := Arbitrary<seq<int>>.Lists(elems, 0, 20);
  var sut := new DedupSut();
  var _ := RunMethodTest(arb, sut, "dedup preserves the set, drops duplicates");
}
```

## Seeded runner (reproducible & fresh-seed runs)

`RunConfig<T>` controls a run:

```dafny
datatype RunConfig<!T> = RunConfig(
  numRuns: nat,                     // examples to attempt
  seed: Option<bv64>,               // None => library default seed (42)
  examples: seq<T>,                 // concrete inputs always tried before random ones
  classifier: Option<T -> string>,  // bucket-label inputs for distribution stats
  useColor: bool,
  verbosity: Verbosity)             // Off | Low | Medium | High

// DefaultConfig<T>() == RunConfig(100, None, [], None, true, Low)
```

Reproducible run with a fixed seed (use `.(field := val)` to override fields):

```dafny
method {:test} TestSeeded() {
  var arb := Arbitrary<int>.Range(0, 100);
  var cfg := DefaultConfig<int>().(seed := Some(123), numRuns := 50, verbosity := Medium);
  var _ := RunTestWithConfig((n: int) => 0 <= n < 100, arb, "config-demo", cfg);
}
```

Fresh random seed each run — from the `SeededTesting` module, which draws a new
seed via `GetSeed()`:

```dafny
include "../src/SeedSource/SeededTesting.dfy"
// import opened SeedSource
// import opened SeededTesting

method {:test} TestRandomSeed() {
  var arb := Arbitrary<int>.Range(0, 1000);
  var _ := RunTestRandom((n: int) => 0 <= n < 1000, arb, "random-seed run");
  // RunTestRandomWithConfig(pred, arb, name, cfg) fills the seed only when
  // cfg.seed is None, so an explicitly-set seed stays reproducible.
}
```

## Shrinking

When a property fails, DafnyCheck **automatically shrinks** the counterexample to
a minimal case (deletion, zeroing, per-value minimization, and swapping) before
reporting — no configuration required. The failing input you see is the shrunk one.

## Pitfalls

1. **`{:test}` attribute placement**: Must be `method {:test} Name()`, never on a separate line before `method`.
2. **LSP false positives**: DafnyCheck imports `Std.Wrappers` which LSP may not resolve but IS available at compile time. Ignore "module Std does not exist" errors from the editor — verify with `dafny.exe`.
3. **Path handling in WSL bash**: Use forward slashes for Windows paths: `"C:/Users/..."` not `"C:\Users\..."`.
6. **Sequence concatenation:** Uses `+`, not `++`.
7. **`Mix` branch disjointness:** every generator in `Mix([...])` must have a disjoint `repr` — build a fresh generator per branch; reusing one `Arbitrary` in two branches fails the precondition.
8. **Generators are methods, not expressions:** bind them first (`var g := Arbitrary<int>.Range(0, 100);`) — you cannot write `Arbitrary<int>.Range(0,100).Map(...)` inline.

## Common Property Patterns

- **Idempotence**: `f(f(x)) == f(x)`
- **No trailing zeros**: `result[|result|-1] != '0' || result == "0"`
- **Length non-increasing**: `|f(x)| <= |x|`
