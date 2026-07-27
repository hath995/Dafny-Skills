---
name: dafny-type-characteristics
description: Use when writing or reading Dafny type parameter characteristics (T(0), T(00)) — auto-initializable and nonempty type constraints on generics.
tags:
  - dafny
  - declaration
  - type
related_skills:
  - dafny-module
  - dafny-function-and-predicate
---

# Dafny: Type Parameter Characteristics

Type parameters can carry characteristics beyond `(==)` and `(!new)`. These control initialization rules and quantifier reasoning.

## Auto-initializable types: T(0)

A type marked `(0)` has a default zero value that Dafny can construct. This is what lets you allocate an array of that type without supplying an element initializer:

```dafny
method WithZero<A(0)>() { var arr := new A[10]; }   // OK — A has a zero value

// error: unless an initializer is provided for the array elements,
//        a new array of 'A' must have empty size
method WithoutZero<A>() { var arr := new A[10]; }
```

## Nonempty types: T(00)

A type marked `(00)` is guaranteed non-empty, but Dafny may not be able to *construct* an inhabitant. `(00)` is weaker than `(0)`: every `(0)` type is `(00)`, not vice versa.

## Definite assignment and these characteristics

Under Dafny 4's **default** (strict) definite-assignment rules, neither characteristic exempts an out-parameter — every out-parameter must be assigned, including ghost ones:

```dafny
// All three are errors under the default rules:
method Demo<A(0), X>() returns (a: A, x: X) { }
method Demo2<B(00), X>() returns (b: B, ghost g: B, ghost h: X) { }
```

The exemptions apply only under `--relax-definite-assignment`. With that flag:

```dafny
// dafny verify --relax-definite-assignment
method Demo<A(0), X>() returns (a: A, x: X)
{
  // OK: 'a' is auto-initialized to its zero value
  // error: out-parameter 'x' has not been given a value
}

method Demo2<B(00), X>() returns (b: B, ghost g: B, ghost h: X)
{
  // error: non-ghost out-parameter 'b' has not been given a value
  // OK: ghost out-parameter 'g' is fine — type B is nonempty
  // error: 'h' has not been given a value (X lacks (00))
}
```

## Notes

- `(==)` requires equality support; `(!new)` requires the type to contain **no references**, so it cannot be instantiated with a class type (`Test<Ref>()` fails with *"type parameter (T) ... must contain no references"*). See dafny-module for abstract module type constraints.
- `(0)` provides auto-initialization — Dafny can synthesize a default value, which is what array allocation without an initializer needs.
- `(00)` guarantees non-emptiness — useful for ghost variables and quantifier reasoning over types known to have inhabitants, but it does not by itself relax definite assignment under Dafny 4 defaults.
- Multiple characteristics combine: `T(==, !new)`, `A(0, ==)`.
