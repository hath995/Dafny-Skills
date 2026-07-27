---
name: dafny-module
description: Use when writing or reading Dafny modules (module, import, export, provides, opened, abstract, refines) or pulling in another .dfy source file (include) — namespaces with controlled exports, refinement, and cross-file includes.
tags:
  - dafny
  - declaration
related_skills:
  - dafny-trait
  - dafny-ghost-and-const
---

# Dafny: Module

Modules group declarations into namespaces with `import`/`export` control and support abstract refinement.

## Syntax

```dafny
module Lib {
  export provides f       // reveal signature only
  function f(x: int): int { x + 1 }
}

module M {
  import Lib              // qualified: Lib.f(3)
}

module N {
  import opened Lib       // unqualified: f(3)
}                         // (the two forms cannot both name Lib in one module)

abstract module Iface {
  method M() returns (r: int)
}

module Impl refines Iface {
  method M() returns (r: int) { r := 42; }
}

// Abstract type with constraints
abstract module Magma {
  type T(==, !new)           // (==) supports equality, (!new) contains no references
  function op(a: T, b: T): T
}

// Module parameterized by imported modules
abstract module Subgroups {
  import G : Group           // G must refine the abstract Group module
  import L : Collection      // L provides collection operations over G.T

  ghost predicate IsSubgroup(s: L.C<G.T>) {
    L.In(G.Id(), s)
  }
}

// Concrete module alias: bind an abstract parameter to a concrete module
module ZModule refines LeftModule {
  import R = IntRing          // R is now the concrete IntRing module
  import V = IntPairGroup     // V is now the concrete IntPairGroup module

  function smul(r: R.T, v: V.T): V.T { V.Add(v, v) }
}

// Combined export with reveals and provides
module SeqFunctions {
  export MapFns reveals Map provides LemmaMapDist, fmap   // reveal bodies + provide signatures
  export reveals *                                          // reveal everything else
}
```

## Including other source files

`include` is a **file-level directive** (not a module construct) that pulls in
another `.dfy` file so its declarations are available. It must appear at the top
of the file, before any declarations, with a path relative to the current file.

```dafny
include "../lib/Math.dfy"
include "helpers/Seqs.dfy"

module Client {
  import opened MathLib     // still need `import` to use a module from Math.dfy
}
```

## Notes

- `include "file.dfy"` makes the file's contents available; it is **not** the
  same as `import` — you still `import` any module you want to use (top-level
  declarations become directly available).
- Include paths are relative to the including file; includes are transitive and
  de-duplicated, so including the same file along multiple paths is harmless.
- `include` directives must precede all other declarations in the file.

- `export provides f` exposes only the name's signature, hiding the body.
- `import opened` drops the module qualifier, bringing names into scope directly.
- `abstract module`s declare interfaces with unspecified bodies.
- A concrete module `refines` an abstract one, filling in the missing definitions.
- `type T(==)` declares an abstract type with equality support; `(!new)` requires the type to contain **no references** — it cannot be instantiated with a class type. Passing one fails with *"type parameter (T) ... must contain no references"*. See dafny-type-characteristics.
- `import G : Group` inside an abstract module makes `G` a module parameter that must refine `Group` at instantiation.
- `import R = ConcreteModule` binds an abstract parameter to a concrete module in a refinement (e.g., `module Impl refines Abstract { import R = IntRing }`).
- `export X reveals A, B provides C, D` combines revealing bodies of `A`, `B` while only exposing signatures of `C`, `D`.
