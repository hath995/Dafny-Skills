---
name: lemmata-provenance
description: Use when deciding which version of a Lemmata package is safe to use or upgrade to, or whether to trust a dependency's proofs — `lem view`, `lem provenance <name> [--use a,b]` (the import-set compatibility window), `lem provenance --closure` (diamond verdicts), `lem audit` (verified .doo vs source, declared unproven obligations, extern capabilities, era check), `lem outdated`, and era-certification refusals from `lem add`/`lock`/`update`. Also use when two dependencies need different majors of one library.
tags:
  - lemmata
  - tooling
related_skills:
  - lemmata-dependencies
  - lemmata-publish
---

# Lemmata: Provenance, eras, and trust

Lemmata records, for every release and every exported symbol, whether the
symbol was **added**, changed **compatibly**, changed in a **breaking** way,
or **removed**. A major version (the compatibility class) bumps when *any*
symbol breaks. Most code uses only a few symbols, so provenance answers the
finer question: is this version compatible with what **this project
actually imports**?

## Looking up a package

```bash
lem view ramdaf             # versions per major + exported modules/symbols (add --json)
lem status ramdaf           # latest release incl. pending/rejected
```

The `(default export)` entries in `lem view` are the module names you
`import`.

## The import-set window

```bash
lem provenance ramdaf                     # import set inferred from this project's source
lem provenance ramdaf --use Ramdaf.List.reduce,Ramdaf.List.mapSeq
```

`--use` takes **fully qualified** names (`Module.Path.symbol`, as `lem view`
prints them). A bare name like `reduce` matches nothing. It is reported as
"used but not present at latest (removed upstream)", and the window silently
widens to the whole timeline. Treat that note as a typo signal, not as
information about the package.

```
project import set (1 symbol(s) of ramdaf used here): Ramdaf.List.reduce
compatible window: >= 16.0.0  &&  <= 35.0.8
minimum supporting release: 16.0.0
```

The window starts at the release where a used symbol last broke (or was
born) and runs to the latest release where all of them still exist. Other
symbols churning across majors don't narrow it. The calculus behind it is
verified. Use the window to answer "can I stay on an older major?" or "is
this upgrade safe for my code?" A manual range such as
`">= 16.0.0 && <= 35.0.8"` in `[dependencies]` works too.

## Diamonds: `--closure`

Dafny links one module namespace per program, so two majors of one library
can't coexist. When two dependers pull in different majors:

```bash
lem provenance --closure
```

```
shared dependency: collections  (2 dependers, major(s) {1,2})
join: RESOLVES — imported symbols don't clash across majors
  -> pin collections to ^2 (2.1.0): one class satisfies all dependers
```

It reports either a single version that satisfies every depender's actual
imports, or the symbol that is needed in two incompatible eras. `lem lock`
already runs this **era repair** automatically. It picks one version per
library, keeps your declared carets binding, and fails only after an
exhaustive verified search proves no era-valid selection exists.

## Era certification on add / lock / update

Every re-lock certifies that each depender's imports are era-consistent at
the selected versions:

```
era check (verified): VALID — every locked edge's imports are era-consistent
  (project) -> ramdaf 35.0.8: ok (imports anchored at 35.0.8)
```

An era-invalid selection is **refused**. The message names the edge and the
symbol. Fix it by changing the version requirement (`lem add <name>@x.y.z`,
or a range), or by updating the depender that is stuck on the old era.
`--allow-era-break` records the selection anyway. Use it only when the user
accepts a known break, never to make the error go away. Path dependencies
and dependencies without served provenance are skipped and reported.

## Trust: `lem audit`

```bash
lem audit [--json]
```

```
trust audit: 1 dependency(ies), 1 verified .doo, 0 from source, 0 declared-unproven obligation(s)
  ramdaf 35.0.8  verified .doo (trusted)
  era check (verified): VALID ...
```

For each locked dependency, the audit reports:
- whether it ships a **verified `.doo`**, re-verified off-box by the
  registry worker and never taken on the uploader's word, or comes **from
  source**, re-verified in your build
- its **declared unproven obligations** (`[verification] expected-errors`),
  i.e. its trust surface. A release that grows this list is forced to a major
  bump.
- the **capabilities** of any shipped non-Dafny extern code (network,
  process, filesystem reach). This is informational only.
- the era check for the whole closure

The registry labels each release with one of these trust levels:

| label | meaning |
| --- | --- |
| verified | ships a trusted `.doo` |
| verified · transitively partial | own code clean, but something it depends on is partial |
| partially verified | verifies except declared obligations. Ships as source |
| source | re-verified in your build |

## Options are part of identity

`standard-libraries`, `unicode-char`, `function-syntax`, and the type-system
edition flags change what a program means. Every package in a closure must
agree on them, so a library built under a different edition is effectively a
different lineage. A conflict surfaces at lock or build time, naming the
option.

## Upgrades

```bash
lem outdated              # newer in-major and new-major versions
lem update                # raise floors within current majors (era-certified)
lem update ramdaf         # also offers newer majors interactively
lem add ramdaf@36.0.0     # cross a major explicitly
```

Before crossing a major, run `lem provenance <name>`. When the window
already covers the new major, nothing you use broke, and the upgrade is a
version edit plus a rebuild.
