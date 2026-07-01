---
name: dafny-module
description: Use when writing or reading Dafny modules (module, import, export, provides, opened, abstract, refines) — namespaces with controlled exports and refinement.
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

## Notes

- `export provides f` exposes only the name's signature, hiding the body.
- `import opened` drops the module qualifier, bringing names into scope directly.
- `abstract module`s declare interfaces with unspecified bodies.
- A concrete module `refines` an abstract one, filling in the missing definitions.
