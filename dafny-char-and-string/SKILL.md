---
name: dafny-char-and-string
description: Use when writing or reading Dafny text types (char, string) — character and string values with indexing and slicing.
tags:
  - dafny
  - type
  - collection
related_skills:
  - dafny-seq
  - dafny-slicing
  - dafny-collection-operators
---

# Dafny: Char and String

`char` is a single character; `string` is a sequence of characters.

## Syntax

```dafny
var c: char := 'A';
var s: string := "hello";

assert s[0] == 'h';       // index a single char
var sub := s[1..3];       // slice, sub == "el"
```

## Notes

- `string` is a synonym for `seq<char>`, so all sequence operations apply.
- Index a single character with `[i]`; slice a substring with `[i..j]` (j exclusive).
- Char literals use single quotes (`'A'`); string literals use double quotes (`"hello"`).
- Concatenate strings with `+` (not `++`).
- `|s|` gives the length.
