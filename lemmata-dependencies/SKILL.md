---
name: lemmata-dependencies
description: Use when a Dafny project uses Lemmata / the `lem` CLI (a `lemmata.toml`, `lemmata.lock`, or `deps/` directory is present) or when you want to use a library from the registry at registry.lemmata.sh — `lem init`, `lem add/remove/update`, `lem build`/`verify`, `lem run`/`test`, `lem task`, importing a dependency's modules from Dafny code, and fixing restore/lock/toolchain errors. Also use for duplicate-module resolution errors after `lem restore --source`, "run lem restore first", stale-lock messages, and a `.doo` that will not load.
tags:
  - lemmata
  - tooling
related_skills:
  - lemmata-manifest
  - lemmata-provenance
  - lemmata-workspace
  - dafny-module
---

# Lemmata: Using dependencies (`lem`)

Lemmata is a package manager and registry for Dafny. The CLI is `lem`. It
bundles its own pinned Dafny + Z3, so a project needs no separately installed
Dafny. `lem doctor` reports the toolchain, registry reachability, and
credentials.

## How to recognise a lem project

| file | owner | commit it? |
| --- | --- | --- |
| `lemmata.toml` | you (and `lem add/remove/update`) | yes |
| `lemmata.lock` | `lem` only. Holds the certified resolution, toolchain, and hashes | yes |
| `dfyconfig.toml` | `lem restore` rewrites `[options].library` | usually yes |
| `deps/` | `lem` only. Holds materialized dependencies | no (`**/deps/` in `.gitignore`) |
| `.lem-publish/` | `lem publish` / `lem pack` staging | no |

Never hand-edit `lemmata.lock`, `deps/`, or the `library = [...]` list in
`dfyconfig.toml`. `restore` prunes anything in `deps/` that the lock does not
name and rebuilds the library list.

## Typical loop

```bash
lem init mylib                 # prompts: Dafny version, then registry metadata (Enter skips)
mkdir -p src                   # init writes lemmata.toml only; sources go under src/
lem view ramdaf                # versions + exported modules/symbols of a package
lem add ramdaf                 # latest major as ^x.y.z, re-lock, restore
lem build                      # alias: lem verify. Restore + verify in dependency order (cached)
lem test                       # compile + run {:test} methods (needs a restored closure)
```

A failed `lem add` rolls back the `lemmata.toml` edit, so the project is left
unchanged.

## Importing a dependency in Dafny code

Do **not** `include` dependency files. `lem restore` puts each dependency's
verified `.doo` in `deps/<name>.doo` and lists it under
`dfyconfig.toml [options] library`. Your code just imports the dependency's
**Dafny module names**. Those are the names `lem view <name>` lists as
`(default export)`, not the package name.

```dafny
// lemmata.toml: [dependencies] ramdaf = "^35.0.8"
module Demo {
  import Ramdaf.List            // module name from `lem view ramdaf`

  function Total(xs: seq<int>): int {
    List.reduce((a: int, b: int) => a + b, 0, xs)
  }
}
```

After a restore, plain `dafny verify dfyconfig.toml` (and the IDE) work too.
`dfyconfig.toml` has no `includes` key, so Dafny defaults to `**/*.dfy` plus
the library list.

## Command reference

| command | does |
| --- | --- |
| `lem add <name>[@x.y.z]` | add/replace a dep at `^x.y.z` (bare = latest major) |
| `lem add <name> --dev` | into `[test.dependencies]`: used by run/test, never shipped |
| `lem add <name> --source` | materialize as source, not `.doo` |
| `lem add --path ../lib` | link a local lem project live (dev only; publish refuses it) |
| `lem remove <name>` | drop and re-lock |
| `lem update [<name>]` | raise caret floors within the current major. A named update offers newer majors interactively |
| `lem outdated` | locked deps with newer versions |
| `lem lock` / `lem restore` / `lem ci` | resolve → `lemmata.lock`; materialize → `deps/`; clean restore from the lock only |
| `lem ls` / `lem why <name>` | the locked closure; what pulls a dep in |
| `lem build [--source]` | verify against the closure; `--source` re-verifies deps from source |
| `lem run [<file>] [-- <args>]` / `lem test [<file>]` | `dafny run`/`test` over the restored closure. Do NOT restore. `-- <args>` needs lem ≥ 0.1.24 |
| `lem task [<name>] [-- <args>]` | list / run the project's `[tasks]` (lem ≥ 0.1.24; see `lemmata-manifest`) |
| `lem cache ls\|clean\|reset` | `reset` also drops `lemmata.lock` + `deps/` |
| `lem toolchain list\|install <v>` | hash-pinned Dafny toolchains |

Every registry-touching command takes `--registry <url>` (default
`https://registry.lemmata.sh`).

## Reading a dependency's source

`lem view <name>` lists its exports. To read the actual code:

```bash
lem restore --source           # deps/<name>/src/*.dfy instead of deps/<name>.doo
# ... read deps/<name>/src/ ...
lem restore                    # back to the verified .doo
rm -rf deps/<name>/            # REQUIRED: restore leaves the source dir behind
```

If you skip that `rm`, `dafny verify dfyconfig.toml` and the IDE see both
`deps/<name>/src/*.dfy` (via `**/*.dfy`) and `deps/<name>.doo`. Every module
of the dependency is then declared twice, and you get a flood of
duplicate-module resolution errors. `lem build` itself is unaffected, so the
breakage shows up only in the editor or with direct `dafny`.

## `run` / `test` specifics

- They are a thin layer: they do not restore or re-lock. If you see "run lem
  restore first", run `lem restore` (or `lem build`).
- They use the test configuration: `[test.options]` over `[options]`, and
  `[test.source].include` files (e.g. `tests/**/*.dfy`) are compiled in.
- The first run prompts for a backend (`cs/java/go/js/py`). It defaults to
  `cs` when non-interactive and is saved as `[package].preferredTarget`.
- `lem test tests/FooTest.dfy` compiles only that file plus the library
  closure.
- Program arguments (lem ≥ 0.1.24) go after `--`:
  `lem run [<file>] -- --in data.txt 3`. `Main(args: seq<string>)` gets the
  program name in `args[0]` and your arguments from `args[1]` on. Everything
  after `--` belongs to the program, so `lem run -- --source` does not switch
  lem to source. lem ≤ 0.1.23 silently drops program arguments.
- `dafny run` prints a compiled program's output only when the program exits,
  so don't rely on seeing progress output live.

## Toolchains and `.doo` loading

- `[toolchain] dafny = ">= 4.10.0"` constrains the Dafny version. lem picks
  `$LEM_DAFNY`, then the bundled Dafny, then an installed lem toolchain,
  then `PATH`. When none satisfies the constraint, it exits and names the
  version to install: `lem toolchain install <v>`.
- A `.doo` loads only under the Dafny that built it. When a dependency's
  release was built by a different Dafny, restore falls back to its
  **source** and announces it. The build then re-verifies that dependency,
  which is slower but correct.

## Troubleshooting

| symptom | fix |
| --- | --- |
| lock is stale / manifest changed | `lem lock` (or `lem build`, which restores) |
| `.doo` hash 404s after a registry reset | `lem cache reset`, then `lem build` |
| add/lock refused as era-invalid | read the message and see `lemmata-provenance`. `--allow-era-break` records a known break; don't use it to silence one |
| own proof fails, unclear which assertion | `lem verify --isolate [--time-limit S]` lists every obligation that does not close (diagnostic only) |
| slow verification | `lem build --profile [--top N]` ranks scopes by solver resource count |
| errors inside a dependency `.doo` | locations point into the serialized program, not source lines. Use `lem restore --source` to read it |
| `lem build` prints "cannot run dafny: Could not find a part of the path '…/src'" and then "build OK" | **not verified.** lem ≤ 0.1.23 `build` only looks in `src/`, so a package whose `to_export` lies elsewhere gets a false green with exit 0. Upgrade to 0.1.24+, or verify with `lem pack` (reads `[source]`, prints the real errors, writes no `.doo` on failure), or move the sources under `src/` |
| `lem doctor` exits on a minimal image | install ICU (`libicu-dev`) |
| IDE / `dafny verify dfyconfig.toml`: "cannot load deps/X.doo: --standard-libraries is set locally to False, but the library was built with True" | an option a `--dev` dependency needs went into `[test.options]`, which `dfyconfig.toml` doesn't carry. `lem build`/`lem test` are correct. Adding the option to `dfyconfig.toml [options]` fixes the editor, but `lem publish` captures `dfyconfig.toml [options]` into the shipped manifest, so remove it before publishing |

`dafny verify` passing does not guarantee `lem pack` / publish passes.
Packing runs `dafny build -t:lib`, which verifies through a different path.
Before publishing a large proof, run `lem pack` (see `lemmata-publish`).
