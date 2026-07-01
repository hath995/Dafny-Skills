---
name: dafny-tuple
description: Use when writing or reading Dafny tuple types ((A, B, ...) with .0/.1 field access) — anonymous positional product types.
tags:
  - dafny
  - type
related_skills:
  - dafny-datatype
  - dafny-primitive-types
---

# Dafny: Tuple

Anonymous product types grouping several values, accessed by position.

## Syntax

```dafny
var t: (int, bool) := (42, true);
assert t.0 == 42 && t.1 == true;

var nested: (int, (bool, char)) := (1, (true, 'x'));
assert nested.1.0 == true;

var (a, b) := t;              // destructure
```

## Notes

- Anonymous positional product types; the type is written `(A, B, ...)`.
- Access components by position: `.0`, `.1`, `.2`, …
- Can be nested (`t.1.0`) and destructured in `var` bindings or match patterns.
- Tuples are value types compared by structural equality.
