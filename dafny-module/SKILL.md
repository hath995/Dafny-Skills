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
  import opened Lib       // unqualified: f(3)
}

abstract module Iface {
  method M() returns (r: int)
}

module Impl refines Iface {
  method M() returns (r: int) { r := 42; }
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
