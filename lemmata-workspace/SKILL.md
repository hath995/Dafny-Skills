---
name: lemmata-workspace
description: Use when a source tree holds several inter-dependent Lemmata packages grouped by a `lem-workspace.toml` — `lem init --workspace`, members and globs, shared [workspace.toolchain]/[workspace.options], how internal edges resolve to sibling .doo files, graph-wide `lem lock/restore/verify/pack/publish` (at the root vs inside a member vs `-w`), incremental reuse (`--force`, `--rebuild`), `--jobs`, and finding the expensive or failing lemma with `--profile` and `--isolate <member>`. Also use when member order looks wrong or a sibling edge is silently missing.
tags:
  - lemmata
  - tooling
related_skills:
  - lemmata-dependencies
  - lemmata-manifest
  - lemmata-publish
---

# Lemmata: Workspaces

A workspace groups several lem packages in one source tree that depend on
each other. The whole graph then locks, verifies, packs, and publishes in
dependency order with one command. It is purely client-side; the registry
never sees it.

## Layout

```
my-project/
  lem-workspace.toml        # the root: NOT a lemmata.toml
  regex-engine/lemmata.toml # members are ordinary packages
  linden-core/lemmata.toml
  linden-equiv/lemmata.toml
```

```toml
# lem-workspace.toml
[workspace]
members = ["regex-engine", "linden-*"]   # member dirs; globs allowed

[workspace.toolchain]
dafny = ">= 4.11.0"          # inherited by members that don't set their own

[workspace.options]
standard-libraries = true    # shared options; a member may override
```

`lem init --workspace` scaffolds this file. Each member is created with
`lem init <name>` in its own directory.

## Internal edges

A member depends on a sibling with an **ordinary registry-style
requirement**, using the sibling's canonical name (dotted when namespaced, no
quotes):

```toml
[dependencies]
linden.semantics-core = "^1.0.0"   # a sibling: resolves to its freshly built .doo
ramdaf = "^35.0.8"                 # external: resolves from the registry
```

- Inside the workspace, the edge resolves to the sibling's **verified
  `.doo`**, rebuilt in dependency order. The `.doo` floats, so edits to the
  sibling are picked up on the next `verify`. Dependents trust the sibling's
  proofs instead of re-verifying its source. Don't `include` sibling files and
  don't use `--path` between members.
- The same line resolves to the registry once that sibling is published, so a
  member's manifest is always publish-ready.
- **Edges are matched by package name, not directory.** A dependency key
  that misses the sibling's current name, e.g. one left over after a rename,
  silently produces no edge, and member order falls back to declaration
  order. Rename manifests and dependency keys in one pass, then run
  `lem lock` at the root.

## Where you run a command matters

| where | `ls` / `lock` / `restore` / `verify` / `pack` / `publish` act on |
| --- | --- |
| workspace root (holds `lem-workspace.toml`) | every member, in dependency order |
| inside a member directory | that one package, as outside a workspace |
| inside a member, with `-w` / `--workspace` | the whole graph (walks up to the root) |

The graph commands are **thin**. They assume each member's lock is current
and its external deps are restored, and they never re-lock or fetch behind
your back. The usual sequence from a fresh checkout:

```bash
lem lock          # at the root: re-lock every member (internal edges float, external pinned)
lem restore       # each member's EXTERNAL deps into its deps/ (siblings are staged by the walk)
lem verify        # dependency-ordered verification; stops at the first failing member
lem pack          # same walk, also writes each member's archive + .doo into .lem-publish/
lem ls            # member graph in dependency order
```

Each member verifies with the toolchain its own `[toolchain].dafny` (or the
workspace's) selects. That is the version publish and the registry worker
will use, not necessarily lem's bundled Dafny.

## Incremental reuse and concurrency

- `verify` and `pack` reuse an unchanged member and print
  `packed <name> (cached, unchanged)`. A member counts as unchanged only when
  every `.dfy` under it, its `lemmata.toml`, the content of each staged
  `.doo`, the option flags, the Dafny version, and lem's build identity are
  all unchanged. A failed walk therefore resumes instead of restarting, and an
  edit rebuilds only that member and its dependents.
- `lem pack --force` rebuilds everything, and `lem pack --rebuild <member>`
  rebuilds that member plus its dependents.
- `--jobs N` builds independent members (same topological level)
  concurrently and divides cores between them. Sequential is the default
  because it has the predictable memory profile; raise N only when the
  machine has the memory for it.

## Finding the slow or failing lemma

```bash
lem pack --profile --top 10                         # per built member: costliest scopes by solver resource count
lem verify --isolate my-member [--time-limit 30]    # build only its deps, then list every obligation that does not close
```

`--isolate` is a diagnostic. It writes no `.doo`, touches no cache, and exits
1 when anything did not close. Don't ship `{:isolate_assertions}` as the fix.
Split the lemma instead (see `dafny-lemma`, `dafny-triggers`).

`dafny verify` passing does not guarantee `lem pack` passes. Pack runs
`dafny build -t:lib`, whose verification path can behave very differently on
a large unsplit lemma (memory blow-ups, timeouts). Check big members with
`lem pack` before publishing.

## Publishing a workspace

At the root, `lem publish` goes **leaf-first**. For each member it waits for
the member's dependencies to be served, re-locks it against the registry
(pinning their freshly published `.doo` hashes), publishes it, and waits
until it is verified before moving to dependents. It stops at the first
failure. One namespace claim covers all members. All the confirmation rules
in `lemmata-publish` apply; a workspace publish uploads many packages.

Mark the members people should use with `entry = true` in their own
`[package]` table (lem ≥ 0.1.25). Registry search lists only those, plus
packages outside any namespace, unless the reader asks for all, so an
unmarked workspace is hidden from default search. A workspace's API packages
are usually its most depended-on members, so nothing can infer them. The flag
goes in each member's manifest; `lem-workspace.toml` is never uploaded. See
`lemmata-manifest`.

## Not supported (v1)

No shared lockfile (each member keeps its own `lemmata.lock`), no
coordinated version bumping, and no nested workspaces.
