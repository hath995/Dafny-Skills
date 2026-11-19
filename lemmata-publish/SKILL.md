---
name: lemmata-publish
description: Use when releasing a Dafny package to the Lemmata registry — `lem login`, `lem pack`, the staged `lem publish` flow (--yes / --allow-major / --cancel), how the registry assigns patch/minor/major from a compatibility proof, namespaces, pending/verified/partial/rejected status via `lem status`, and pre-flight checks. Also use when publish aborts on verification warnings, warns about {:test} methods or unused dependencies, refuses a path dependency, or reports obligations "NOT proved".
tags:
  - lemmata
  - tooling
  - release
related_skills:
  - lemmata-compat-bridges
  - lemmata-manifest
  - lemmata-workspace
---

# Lemmata: Publishing (`lem publish`)

## Ground rules for agents

- **Publishing is outward-facing and permanent.** Releases cannot be
  yanked, and a namespace claim is permanent. Run the pre-flight checks and
  the staging step freely. Get the user's explicit go-ahead before any
  command that uploads (`--yes`, answering `y`), and separately before
  `--allow-major`.
- `lem login` is interactive (GitHub device flow). Ask the user to run it
  themselves, e.g. `! lem login`. Check with `lem whoami`.
- **You don't choose the version.** The registry derives it from a
  compatibility check against the predecessor and writes it back to
  `[package].version`. The first release of a name is `1.0.0`.

## Tiers: how the version is decided

On publish, the previous release is materialized as `module Old { ... }`. A
proof obligation is generated for every changed contract, and Z3 decides the
tier. The tier is about who does the proving:

| tier | condition | version |
| --- | --- | --- |
| patch | every obligation discharged with an **empty body** | `c.m.p+1` |
| minor | proved, but some obligation needed an author `_Manual` bridge lemma, or a dependent needs a hint (e.g. `reveal` after a function became opaque) | `c.m+1.0` |
| major | a refuted/unproved obligation, or a structural change: signature or type change, removal, datatype constructor change, trust-base growth (new `{:axiom}`, larger `expected-errors`), ghost/compiled phase flips | `c+1.0.0` |

What follows from the table:
- **Pure additions are patch**, because they create no obligations for
  existing dependents.
- Weakening a `requires` or strengthening an `ensures` is compatible when
  provable. Strengthening `requires`, weakening `ensures`, or changing a
  transparent function's value is breaking.
- Rewriting a function or predicate body to an equivalent one is compatible
  only when the equivalence is proved. When it needs induction it fails as
  emitted, and you write a bridge (`lemmata-compat-bridges`).
- **Never advertise unproven compatibility.** When the proof is out of
  reach, the release is major. That is by design, not a bug to work around.
- A major is a new compatibility class (`name@N`). Consumers on `^old` never
  float into it.

## Pre-flight (all local, safe to run)

```bash
lem build                 # must be clean against the locked closure
lem pack                  # builds the publishable archive + .doo exactly as publish will
```

Publish **aborts** when, or warns that:
- **Warnings in the package's own source abort it.** The registry serves
  only warning-free libraries. Fix each warning; a typical one is a missing
  trigger, fixed with an explicit `{:trigger}`, or `==>` precedence fixed
  with parentheses. Don't suppress warnings with options.
- **A `[dependencies]` path dep is refused.** Publish the target first, then
  `lem add <name>`. Path deps under `[test.dependencies]` are fine.
- **It warns about `{:test}` / `@Test` methods in shipped source.** Move
  them under `[test.source]`.
- **It warns about a declared dependency proven unused.** Remove it from
  `lemmata.toml`.
- **It warns when `CompatProofs.dfy` has no lemma matching this release's
  obligations.** That file is ignored for this release.
- `[tasks]` (lem ≥ 0.1.24) is local: publish strips it from the uploaded
  manifest, which the registry otherwise serves publicly.

`dafny verify` passing does not guarantee `lem pack` passes. Pack runs
`dafny build -t:lib`, which verifies through a different path and can blow
up on a large unsplit lemma. The registry worker kills a build at about
15 minutes, and `verification-time-limit` is capped at 900.

## The staged flow

```bash
lem publish               # stage: run compat locally, print the tier/version, prompt
lem publish --yes         # approve + upload (patch/minor; a major still needs --allow-major)
lem publish --yes --allow-major   # consent to a breaking class bump
lem publish --cancel      # discard .lem-publish/ staging; CompatProofs.dfy is kept
```

1. `lem publish` resolves the predecessor, builds the compat artifact under
   `.lem-publish/`, and verifies it locally with the **selected** toolchain.
2. When obligations are unproved, it prints each one with a hint and writes
   the goals to `.lem-publish/obligations.dfy`. It scaffolds `_Manual` stubs
   into `CompatProofs.dfy` at the package root, or into
   `.lem-publish/CompatProofs.stubs.dfy` when that file already exists.
   Non-interactive runs stop there with guidance and do not upload.
3. Then you choose:
   - Prove the stubs and re-run `lem publish`. The predecessor stays cached
     in `.lem-publish/`. Done this way the release becomes minor.
   - Accept the break with `--allow-major` (only with user consent).
   - Change the code so the obligation goes away.
4. Upload sends **source only**. The registry's off-box worker re-verifies
   with the same Dafny version. Until then the release is **pending**, and
   the client's `.doo` is never trusted.

After a successful upload, commit `lemmata.toml` (it now holds the new
version) and `CompatProofs.dfy` if present. The next release needs that
file.

## After upload

```bash
lem status mylib            # latest release incl. pending/rejected (full dotted name if namespaced)
lem view mylib              # served versions + exports
```

| outcome | meaning |
| --- | --- |
| **verified** | clean. Ships a trusted `.doo` |
| **verified · transitively partial** | own code clean, but a dependency is partially verified |
| **partially verified** | verifies except the declared `expected-errors`. Ships as source |
| **rejected** | the worker's rebuild failed, e.g. a warning or a timeout. Fix it and publish again |

## Namespaces

`[package] namespace = "acme"` makes the identity `acme.mylib`. The first
publish into an unowned namespace claims it for your account. Reserve one
ahead of time with `lem namespace claim acme`, and list them with
`lem namespace list`. Both are permanent and first-come, so confirm the
name with the user.

## Workspaces

At a `lem-workspace.toml` root, `lem publish` publishes every member
leaf-first. Each member is re-locked against the registry, published, and
awaited until verified before its dependents run, stopping at the first
failure. See `lemmata-workspace`.
