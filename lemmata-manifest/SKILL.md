---
name: lemmata-manifest
description: Use when reading or editing a Lemmata `lemmata.toml` manifest — [package] (name, namespace, version, entry), [toolchain], [source] to_export / not_exported, [options] (semantic Dafny flags, verification-time-limit, cores), [verification] expected-errors, [dependencies] (caret, path, namespaced dotted keys), the [test.*] mirror, and [tasks] (named `lem task` command lines). Also use to decide what belongs in [source] vs dfyconfig.toml includes, or for manifest errors such as a rejected option, an options conflict across the closure, or a package name/namespace that is refused.
tags:
  - lemmata
  - tooling
  - configuration
related_skills:
  - lemmata-dependencies
  - lemmata-publish
  - lemmata-workspace
---

# Lemmata: `lemmata.toml`

Every Lemmata package has a `lemmata.toml` at its root. `lem init` scaffolds
it; from 0.1.24 it also asks which files ship (`[source]`) and puts unexported
test folders under `[test.source]`. `lem add/remove/update` maintain the
dependency tables. Edit the other sections by hand.

**lem rewrites this file.** `lem add`, `remove`, `update` and `publish`
regenerate `lemmata.toml` from lem's model of it. That drops every
comment and every table lem doesn't know. Keep notes in a README, not in
comments here. On lem ≤ 0.1.23 this includes `[tasks]`, so don't add one
before your lem supports it (below).

## Annotated example

```toml
[package]
namespace = "acme"            # optional; identity becomes acme.mylib
name = "mylib"                # [a-z][a-z0-9-]*, unique on the registry
version = "1.0.0"             # assigned by the registry on publish (see below)
description = "A verified widget library"
license = "MIT"
keywords = ["widgets", "verified"]
entry = true                  # lem >= 0.1.25: an entry point of its namespace
# repository = "..."  homepage = "..."
# preferredTarget = "cs"      # written by the first `lem run`/`test`

[toolchain]
dafny = ">= 4.10.0"           # version formula over Dafny versions

[source]
to_export    = ["src/**/*.dfy"]       # the SHIPPED fileset = package identity + hash
not_exported = ["src/**/*Demo.dfy"]   # subtracted from to_export; excludes win

[options]
standard-libraries = true     # semantic flags: travel with the .doo, bind consumers
verification-time-limit = "300"   # verify-tuning, seconds (1-900); not a consumer contract
cores = "8"                   # verify-tuning, local only

[verification]
expected-errors = []          # declared trust surface; empty = fully verified

[dependencies]
ramdaf = "^35.0.8"                    # registry dep, caret window within one major
linden.equiv = "^1.0.0"               # namespaced: dotted key, NO quotes
mylocal = { path = "../mylocal" }     # local link; publish refuses it here

[test.source]                 # test/dev only: never shipped, never in a consumer's closure
include = ["tests/**/*.dfy"]
[test.options]                # layered over [options] for run/test (test wins)
[test.dependencies]           # dev-only deps (`lem add --dev`)

[tasks]                       # lem ≥ 0.1.24: `lem task <name>`, local only
bench = "run src/Bench.dfy --"
ci    = ["build", "test", "pack"]
```

## Section rules that bite

**`[package].version`.** The registry assigns versions from a compatibility
check, not from you. The first release is `1.0.0`. After that, the bump tier
(patch/minor/major) is derived on publish and written back into this field.
Don't bump it by hand to "request" a version. The major is the
**compatibility class** and part of the package's logical name.

**`[package].namespace`.** The first publish into a namespace claims it,
first-come and permanently. Only the owner may then publish inside it. A
namespace may not share a name with a package. Neither a name nor a namespace
may be `path` or a Windows device name (`con`, `nul`, `aux`, `prn`, `com1-9`,
`lpt1-9`). Use the full dotted name with `lem status`, `lem view`, and
`lem add`: `lem status linden.equiv`.

**`[package].entry`** (lem ≥ 0.1.25; `true` or `false`, default false). Marks a
namespaced package as one people should use directly, not only a building
block for its siblings. Registry search and browse list only **top-level**
packages by default: those outside any namespace plus the namespaced ones with
`entry = true`. The rest of a namespace appears only under "All packages". The
flag is also listed on the namespace card and in `lem namespace list`. Nothing
is inferred, so set it on every member a consumer should start from. A
namespace that marks none shows up in default search only as its card. lem ≤
0.1.24 drops the key whenever it rewrites the manifest, and a non-boolean value
is an error.

**`[source]` vs `dfyconfig.toml`.** These are opposites:
- `to_export` / `not_exported` decide what ships: the archive, the `.doo`,
  and the fileset hash. `deps/` is always excluded. `CompatProofs.dfy` at the
  root also ships (see `lemmata-compat-bridges`), but it is never part of the
  build.
- `dfyconfig.toml` is Dafny's working set for the compiler and IDE. lem
  writes no `includes` key there, so Dafny sees `**/*.dfy`: your sources,
  tests, and deps. lem only manages its `[options].library` list.
- Tests belong under `[test.source]`, not `to_export`. A consumer's
  `lem test` runs every `{:test}` in the compiled program, dependencies'
  tests included, and `lem publish` warns about tests in shipped source.
- With lem ≤ 0.1.23, keep shipped sources under `src/`. Those versions'
  `lem build` verifies only `src/`, so with any other `to_export` it reports
  "build OK" without verifying anything, and `lem pack` is the real check.
  0.1.24 verifies the `[source]` fileset.
- Only the matched files are uploaded, and the registry re-verifies from that
  archive alone. Every file a shipped file `include`s must therefore itself
  be matched by `to_export`.

**`[options]`** accepts an **allowlist** of globally coherent semantic flags:
`standard-libraries`, `unicode-char`, `function-syntax`, the type-system
edition flags (`type-system-refresh`, `general-traits`, `general-newtypes`),
and similar. Arbitrary `dafny` flags are rejected, and so are `--boogie`
options. The build takes the **union** of the options of the project and
every dependency. One name bound to two values anywhere in the closure is a
conflict. A dependency's options propagate to you, and `lem add` may offer to
adopt them. `lem publish` captures `dfyconfig.toml [options]` into the
manifest and rejects test/runtime-only options there. The two verify-tuning
keys are special:
- `verification-time-limit` reaches the registry worker. Keep it at 900 or
  below, because the registry rejects more and the worker kills a build at
  about 15 minutes.
- `cores` is purely local. Lower it to stop a memory-heavy build from
  thrashing.

**`[verification] expected-errors`** declares obligations the package
deliberately leaves unproven. Each entry is `"<relpath>: <message>"`, with
line and column stripped and the path relative to the package root:

```toml
[verification]
expected-errors = [
  "src/Arbitrary.dfy: cannot prove termination; try supplying a decreases clause",
]
```

`lem build` accepts a package only when its actual errors equal this
multiset exactly. A new error fails the build as a regression, and a
declared error that no longer occurs fails it as a stale declaration. The
list is trust surface: consumers see it in `lem audit`, and a release that
grows it is forced to a major bump, exactly like adding an `{:axiom}`. Only
use it for a genuine, documented gap, never to get a red build through.

**`[dependencies]`** values are `"^x.y.z"` (a window inside one major) or
`{ path = "..." }`. A path entry takes no version; the target's manifest is
authoritative. Path deps may not nest. Explicit ranges such as
`">= 1.2.0 && <= 1.4.0"` also work. Put namespaced keys unquoted
(`linden.equiv = ...`): TOML nests them and lem flattens them back.

## `[tasks]` and `lem task` (lem ≥ 0.1.24)

Named `lem` command lines, like `npm run` or `deno task`, except every step
is a `lem` command. Check that your lem has it with `lem help task`; older
versions say "unknown command 'task'" and delete the table on the next
rewrite.

```toml
[tasks]
bench        = "run src/Bench.dfy --"         # one command: the arguments after `lem`
ci           = ["build", "test", "pack"]      # in order; stops at the first failure
"fuzz:long"  = "test tests/Fuzz.dfy"          # ':' in a name means a quoted key
```

```bash
lem task                  # list the tasks
lem task ci               # each step echoed ("> lem build") and run as its own lem process
lem task bench -- 1000    # args after -- go on the LAST step: lem run src/Bench.dfy -- 1000
```

- **No shell.** A command is split on whitespace with `'...'`/`"..."`
  quoting; there are no pipes, `&&`, variables or globs. A leading `lem` is
  allowed. For shell work, use a script outside lem.
- **End a task in `--`** when callers should pass program arguments
  through to `Main`. Arguments are appended verbatim to the last step.
- A task may not run another task. Compose with a sequence instead.
- `lem task` exits with the failing step's exit code.
- Names use letters, digits, `_`, `:` and `-`, starting with a letter or
  digit.
- **Local only, like `[test.*]`.** Only the project in the current directory
  runs its tasks. A dependency's `[tasks]` is never listed or run, nothing
  runs a task implicitly (there are no install hooks), and `lem publish`
  strips the table from the uploaded manifest. A workspace root without its
  own `lemmata.toml` has no tasks yet.

## Reserved / rejected

`[features]`, `[provides]`, `[variables]` and a `[targets]` table are
reserved and rejected. (`targets` / `preferredTarget` inside `[package]` are
fine; they serve FFI libraries.)

## Workspaces

A multi-package tree has a separate `lem-workspace.toml` at its root (see
`lemmata-workspace`). Members keep ordinary `lemmata.toml`s and inherit
`[workspace.toolchain]` / `[workspace.options]` when they don't set their own.
